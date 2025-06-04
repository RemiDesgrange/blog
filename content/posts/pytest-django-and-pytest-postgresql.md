---
title: "Setup pytest-django with pytest-postgresql"
date: 2025-05-22T09:50:20+02:00
draft: false
tags: ["Dev", "Pytest", "Django", "Postgresql", "pytest-postgresql", "pytest-django"]
category: ["Testing"]
image: ""
description: "I Finally found out how to setup pytest-django and pytest-postgresql"
lang: "en"
---

At [work](https://ielo.net) we have a pretty standard stack for the backend. A `Django` monolith app with relationnal DB, `PostgreSQL`.

When revamping our test infrastructure, I traded reliablity for speed, using `sqlite` with `spatialite`. Setup of `spatialite` on macOS can be quite cumbersome to make it work with python (I'm skipping this part here).

The setup turn out to be not so simple because I wanted:

* setup of 3 DB.
* fast setup/teardown.

It turns out pytest-postgresql, which will starts a postgresql [cluster](https://www.postgresql.org/docs/current/creating-cluster.html) on your machine, is way faster than using [testcontainer](https://testcontainers.com/).

⚠️ I tried to setup this with ChatGPT/Claude/Mistal and they **all** failed miserably.

## Setup the cluster

```python
# conftest.py

postgres_options = (
        "-c wal_level=minimal "
        "-c max_wal_senders=0 "
        "-c fsync=off "
        "-c synchronous_commit=off "
        "-c full_page_writes=off "
        "-c checkpoint_timeout=1d "  # Long timeout to avoid checkpoints
        "-c max_wal_size=10GB "  # Large max WAL size to avoid forced checkpoints
        "-c shared_buffers=256MB "  # Increased shared buffers for better caching
        "-c work_mem=64MB"  # Larger work memory for faster operations
    )
postgresql_proc_custom = factories.postgresql_proc(postgres_options=postgres_options, dbname="default")
```

You can then override the `django_db_setup` which is explained in the [doc](https://pytest-django.readthedocs.io/en/latest/database.html#advanced-database-configuration). **BUT** there were several culprit:

* You need to delete the existing connection cache (`del connections.__dict__["settings"]`) in order for the fixture to work properly. Otherwise it'll ignore your settings. Note that I tried to use the `django_db_modify_db_settings` fixture without any luck.
* `yield request.getfixturevalue("django_db_setup")` is mandatory at the end of the fixture declaration. If not, setup breaks.

```python
@pytest.fixture(scope="session")
def django_db_setup(request, postgresql_proc_custom):
    from django.conf import settings

    # remove cached_property of connections.settings from the cache
    del connections.__dict__["settings"]
    db_fixture_mapping = {
        "default": postgresql_proc_custom,
        "gis": postgresql_proc_custom,
        "othergis": postgresql_proc_custom,
    }
    for db, fixture in db_fixture_mapping.items():
        settings.DATABASES[db]["ENGINE"] = "django.contrib.gis.db.backends.postgis"
        settings.DATABASES[db]["NAME"] = db
        settings.DATABASES[db]["USER"] = fixture.user
        settings.DATABASES[db]["PASSWORD"] = fixture.password
        settings.DATABASES[db]["HOST"] = fixture.host
        settings.DATABASES[db]["PORT"] = fixture.port
        settings.DATABASES[db]["OPTIONS"] = {"connect_timeout": 10}
        settings.DATABASES[db]["TEST"]["NAME"] = db

    # re-configure the settings given the changed database config
    connections.settings = connections.configure_settings(settings.DATABASES)
    # open a connection to the database with the new database config
    connections["default"] = connections.create_connection("default")
    yield request.getfixturevalue("django_db_setup")
```

Note that sequences will not be reset between tests, you'll need the `django_db_reset_sequence` which is painfully slow. One another good reason to use UUID 😅.
When running with the `transaction=True` argument in `django_db` marker, `TransactionTestCase` will try to truncate all of your tables, but it will **not** `TRUNCATE TABLE <table> CASCADE` unless you specified all of the `available_apps`. So I copy/paste the truncate code from django and hardcoded the `allow_cascade` argument.

```python
@pytest.fixture(scope="session", autouse=True)
def patch_django_truncate():
    """Patch Django's truncate to use CASCADE for PostgreSQL."""

    original_flush = TransactionTestCase._fixture_teardown  # type: ignore

    # literal copied from https://github.com/django/django/blob/f5772de69679efb54129ac1cbca3579b512778af/django/test/testcases.py#L1224
    def patched_fixture_teardown(self):
        # Allow TRUNCATE ... CASCADE and don't emit the post_migrate signal
        # when flushing only a subset of the apps
        for db_name in self._databases_names(include_mirrors=False):
            # Flush the database
            inhibit_post_migrate = (
                self.available_apps is not None
                or (  # Inhibit the post_migrate signal when using serialized
                    # rollback to avoid trying to recreate the serialized data.
                    self.serialized_rollback and hasattr(connections[db_name], "_test_serialized_contents")
                )
            )
            call_command(
                "flush",
                verbosity=0,
                interactive=False,
                database=db_name,
                reset_sequences=False,
                allow_cascade=True,
                inhibit_post_migrate=inhibit_post_migrate,
            )

    TransactionTestCase._fixture_teardown = patched_fixture_teardown  # type: ignore
    yield
    TransactionTestCase._fixture_teardown = original_flush  # type: ignore
```

## Setup db which is not managed

One of our DB is an external gis db, not managed by django. You'll still want to create these models with django for testing purposes.
The following fixture will execute only if one of the test in the test suite require the `custom_not_managed_db`

```python
@pytest.fixture(scope="session", autouse=True)
def create_not_managed_tables(request, django_db_setup, django_db_blocker):
    # Check if any test in the session uses custom_not_managed_db
    needs_custom_not_managed_db = False

    # Iterate through all test items in the session
    for item in request.session.items:
        # Check the item itself and all its parent nodes (class, module)
        for node in item.listchain():
            django_db_marker = node.get_closest_marker("django_db")
            if django_db_marker:
                databases = django_db_marker.kwargs.get("databases", None)
                if databases == "__all__" or (databases and "custom_not_managed_db" in databases):
                    needs_custom_not_managed_db = True
                    break
        if needs_custom_not_managed_db:
            break
    # Only create tables if at least one test needs the not_managed_db
    if needs_custom_not_managed_db:
        with django_db_blocker.unblock():
            from django.apps import apps
            from django.db import connections

            # filter needed
            not_managed_models = filter(lambda m: m.__name__.startswith("CustomeModels"), apps.get_models())
            not_managed_db_connection = connections["not_managed_db"]
            with not_managed_db_connection.schema_editor() as schema_editor:
                for model in not_managed_models:
                    model._meta.db_table = model._meta.db_table.rsplit(".", 1)[-1].strip('"')
                    schema_editor.create_model(model)
```

This journey was quite interesting since I needed to deep dive into the `pytest-django` code base. This setup have added an overhead to my pytest sessions by the order of 1 to 2 seconds on mac M1.

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
* reset of all sequence after each tests. Yes, reling on hardcoded `id`'s in tests are bad, but anyway.
* fast setup/teardown.

⚠️ I tried to setup this with ChatGPT/Claude/Mistal and they **all** failed miserably.

## Setup 3 DB

```python
# conftest.py

def create_postgis_db(host, port, user, dbname, password):
    with (
        psycopg.connect(
            user=user,
            password=password,
            host=host,
            port=port,
        ) as con,
    ):
        con.autocommit = True
        with con.cursor() as cur:
            cur.execute("CREATE DATABASE gis;")
            cur.execute("CREATE DATABASE othergis;")
    for db in ('gis', 'othergis'):
        with psycopg.connect(user=user,
            password=password,
            dbname=db,
            host=host,
            port=port,) as con,
            con.cursor() as cur:
                cur.execute("CREATE EXTENSION postgis;")

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
postgresql_proc_custom = factories.postgresql_proc(postgres_options=postgres_options, load=[create_postgis_dbs], dbname="default")
```

With this setup, I am creating 3 DBs in a single ["cluster"](https://www.postgresql.org/docs/current/creating-cluster.html) to avoid spawning 3 ones.

You can then override the `django_db_setup` which is way explained in the [doc](https://pytest-django.readthedocs.io/en/latest/database.html#advanced-database-configuration). **BUT** there were several culprit:

* You need to delete the existing connection cache (`del connections.__dict__["settings"]`) in order for the fixture to work properly. Otherwise it'll ignore your settings. Note that I tried to use the `django_db_modify_db_settings` fixture without any luck.
* `yield request.getfixturevalue("django_db_setup")` is mandatory at the end of the fixture declaration. If not, setup breaks.

```python
@pytest.fixture(scope="session")
def django_db_setup(request, postgresql_proc_default, postgresql_proc_gcgis, postgresql_proc_liazogis):
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
        settings.DATABASES[db]["NAME"] = fixture.dbname
        settings.DATABASES[db]["USER"] = fixture.user
        settings.DATABASES[db]["PASSWORD"] = fixture.password
        settings.DATABASES[db]["HOST"] = fixture.host
        settings.DATABASES[db]["PORT"] = fixture.port
        settings.DATABASES[db]["OPTIONS"] = {"connect_timeout": 10}
        settings.DATABASES[db]["TEST"]["NAME"] = fixture.dbname

    # re-configure the settings given the changed database config
    connections.settings = connections.configure_settings(settings.DATABASES)
    # open a connection to the database with the new database config
    connections["default"] = connections.create_connection("default")
    yield request.getfixturevalue("django_db_setup")
```


One this that is then left is the reset of sequence after each tests. Since `Django` is known for being extremely bad at database (it matures, but still...) resetting sequences with the integrated `TransactionTestCase.reset_sequence` was slow. My tests where running 10x slower. So I created a little `autouse` fixture to reset sequence between each tests. Didn't test if calling native postgresql table (`pg_namespace`) was faster or not.

⚠️ this works **only** because each tests runs in a transaction which gets rollback at the end of each tests. This wouldn't work without transaction.

```python
@pytest.fixture(autouse=True)
def reset_sequences(postgresql_proc_custom):
    """Fixture to reset all sequences after each test."""
    yield  # This allows the test to run first

    with (
        psycopg.connect(
            dbname=postgresql_proc_default.dbname,
            user=postgresql_proc_default.user,
            password=postgresql_proc_default.password,
            host=postgresql_proc_default.host,
            port=postgresql_proc_default.port,
        ) as con,
        con.cursor() as cursor,
    ):
        # Get all sequences in the database
        cursor.execute("""
            SELECT sequence_name
            FROM information_schema.sequences
            WHERE sequence_schema = 'public'
        """)
        sequences = cursor.fetchall()
        # Reset each sequence to 1
        for sequence in sequences:
            sequence_name = sequence[0]
            cursor.execute(f"ALTER SEQUENCE {sequence_name} RESTART WITH 1;")
```

This setup have added an overhead to my pytest sessions by the order of 1 to 2 seconds on mac M1.

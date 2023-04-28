---
title: "Python Spatialite on Apple Silicon with Pyenv"
date: 2023-04-28T18:46:51+02:00
draft: false
tags: ["sqlite", "spatialite", "apple", "Apple Silicon", "mac", "python", "pyenv"]
category: ["Database", "SQL"]
image: ""
description: ""
lang: "en"
---

For rapid prototyping or for running simple test cases in [Django](https://www.djangoproject.com/) I like to use [spatialite](https://www.gaia-gis.it/fossil/libspatialite/index) the geospatial extension to sqlite.

To use Spatialite, you need to have SQLite compiled with extension support. However, both the built-in SQLite on macOS and the one from Homebrew lack this support. As a result, the Python version compiled by Pyenv also lacks this support, since it compiles a near-default configuration of Python, with SQLite extensions disabled by default.

## Recipe

* First, uninstall sqlite _if you installed it first_

```bash
brew uninstall sqlite
```

Note that if you have package dependent from sqlite you can force the uninstall since we are re-installing sqlite.

* Reinstall sqlite from source (also install spatialite if not installed)

```bash
brew install --build-from-source sqlite
```

* Build python with support for sqlite extension. Here I'm building python 3.10.9 which I did not had on my machine.

```bash
PKG_CONFIG_PATH="/opt/homebrew/opt/sqlite/lib/pkgconfig" \
CPPFLAGS="-I/opt/homebrew/opt/sqlite/include" PATH="/opt/homebrew/opt/sqlite/bin:$PATH" \
LDFLAGS="-L/opt/homebrew/opt/sqlite/lib" \
PYTHON_CONFIGURE_OPTS="--enable-loadable-sqlite-extensions" \
pyenv install 3.10.9
```

* Create a brand new env with spatialite support

```bash
pyenv virtualenv 3.10.9 venv-with-spatialite
```

* Then test it in a new proj

```bash
mkdir test-spatialite && cd test-spatialite
cat venv-with-spatialite > .python-version
```

```python
import sqlite3
from contextlib import closing
# Connect to a Spatialite database
with closing(sqlite3.connect(':memory:')) as conn:
    with conn:
        conn.enable_load_extension(True)
        conn.load_extension('mod_spatialite')
        # Create a table with a geometry column
        with closing(conn.cursor()) as cursor:
            cursor.execute('CREATE TABLE my_table (id INTEGER PRIMARY KEY, geom POINT)')
            # Insert a POINT geometry
            cursor.execute("INSERT INTO my_table (geom) VALUES (GeomFromText('POINT(1 1)'))")
            # Convert the result set to GeoJSON
            cursor.execute('SELECT id, AsGeoJSON(geom)::json FROM my_table')
            rows = cursor.fetchall()
            print(rows)
```



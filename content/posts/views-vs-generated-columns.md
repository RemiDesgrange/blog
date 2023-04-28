---
title: "Views vs Generated Columns"
date: 2020-04-23T14:59:17+02:00
draft: false
tags: ["SQL", "Postgres"]
category: ["SQL", "Postgres"]
---

We are going to talk about SQL Views and Generated Columns in postgresql.

## Introduction

### Views

Have you heard about views in SQL? Maybe! Since it has been here since approximatly forever.

For those who don't what a view is, here is a example :

```SQL
CREATE TABLE person (
    p_id uuid primary key,
    p_firstname text,
    p_lastname text,
    p_whatever_you_need_to_know text,
);

CREATE VIEW v_person AS SELECT p_firstname, p_lastname FROM person;
```

So views are basically a `SELECT`.

On postgresql[^1] you also have `MATERIALIZED VIEW` which are compute when you create the view, not when you request

[^1]: On other DB too ! 

```SQL
CREATE TABLE person (
    p_id uuid primary key,
    p_firstname text,
    p_lastname text,
    p_salary int,
);

-- here insert an insane number of person into the db so any request on it will take > 1sec

CREATE VIEW v_person AS SELECT count(p_id), sum(p_salary), avg(p_salary),  stddev(p_salary) FROM person;
-- it should be long
SELECT * FROM v_person;

CREATE MATERIALIZED VIEW vm_person AS SELECT count(p_id), sum(p_salary), avg(p_salary),  stddev(p_salary) FROM person
-- it should be instant
SELECT * FROM vm_person;
```

Both are great, you just choose **where** computation takes place. That's the beauty of it. 

### Generated Columns

[Generated Columns](https://www.postgresql.org/docs/12/ddl-generated-columns.html) have been introduced in postgresql 12.

They let you have a column computed at insert/update time. Not at query time.

```SQL
CREATE TABLE person (
    p_id uuid primary key,
    p_firstname text,
    p_lastname text,
    p_hashcode text generated always as (sha256(decode(p_firstname || p_lastname, 'escape'))) stored
    );
```

I'm a geo guy, so let's do some geo-usefull stuff with generated column that we usually do via views and/or trigger :

```SQL
CREATE TABLE trip (
    t_id uuid primary key,
    t_date timestamptz,
    t_geom geometry(LineString, 2154),
    t_distance float generated always as (st_length(t_geom)) stored
);

INSERT INTO trip (t_id, t_date, t_geom) VALUES ('bc2c2ec4-a4c3-4df8-922c-b17277e8eb03', now(), 'LINESTRING(0 0, 1 1, 2 1, 2 2)')
SELECT * FROM trip; 
                 t_id                 |            t_date             |          st_astext          |    t_distance     
--------------------------------------+-------------------------------+-----------------------------+-------------------
 bc2c2ec4-a4c3-4df8-922c-b17277e8eb03 | 2020-05-01 10:00:35.580758+00 | LINESTRING(0 0,1 1,2 1,2 2) | 3.414213562373095
```

#### You cannot do everything

Be careful. function used in generated column need to be `IMMUTABLE` !

For example this code cannot work, because `now()` function is not immutable.

```SQL
CREATE TABLE person (
    p_id uuid primary key,
    p_firstname text,
    p_lastname text,
    p_created_or_updated_at timestamptz generated always as (now()) stored
    );
```

## Conclusion


With postgresql release 12, you have more control than ever to choose when you pay the price for data computation. At insert/update or at query time.
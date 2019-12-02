---
title: "Right Management in Postgresql"
draft: false
date: 2017-11-28T10:29:57+01:00
tags: ["Postgresql", "Database", "Right management"]
categories: ["Postgresql", "English"]
---

Recently I though about (re)starting a blog, but didn't find subject to fill it. Then I got a new job with tons of news subject to learn and I need a place to store it, maybe it will help some. Enjoy.

# Right Management in Postgresql

[@Fibrea](https://twitter.com/Fibrea73) we use postgresql a lot, it's the heart of our system. Operating this DB  day to day is quite challenging some time. So here is what I understand from my experience and reading on right management in postgresql.


# Roles, User, Group

On postgresql there is only roles, a user is a role, and a group is role. `CREATE USER x ` is equal to `CREATE ROLE x LOGIN`. You can create two users (so two roles), let's say `bob` and `alice`, and then do `GRANT alice to bob`. There is also inheritence, but the [manual is explaining it well](https://www.postgresql.org/docs/9.4/static/sql-createrole.html)

```SQL
CREATE ROLE bob LOGIN; -- this role can login
CREATE USER bob; -- this role too
CREATE ROLE admin SUPERUSER; -- this role can't login but you can grant permission to it
GRANT admin to bob -- bob gets right from admin; 
CREATE ROLE dev NOCREATEDB NOCREATEROLE NOREPLICATION; -- this role can't login and has no right when created. notice the NO* stuff 
CREATE USER john;
GRANT dev to john; -- grant dev right to john. This will impact john's right.
CREATE TABLE test (id uuid PRIMARY KEY,  data INT NOT NULL); -- create a dummy table
GRANT PRIVILEGES SELECT, INSERT, UPDATE, DELETE ON TABLE test to dev;
SET ROLE john;
-- you're now john
SELECT * FROM test; -- this works. 
INSERT INTO test (id, data) VALUES ('b68baba7-60e7-4cc5-9ae4-c8c38dfd3698', 100); -- this works too
```

# Giving Grant to what

On postgresql there are strange behavior (at first) if you want to control what happend in your DB. You can totaly grant all privileges to a super admin and using it everywhere, but for real life case that's usualy not the best option.


## Grant does not propagate

Imagine the user john created earlier. You can grant him all privileges on a table (or schema) :

```SQL
-- you are postgres or a superuser
CREATE SCHEMA test;
CREATE TABLE test.test_table1 (id uuid, data1 int NOT NULL);
```

```SQL
CREATE USER bob; -- if you didn't do this already
GRANT ALL PRIVILEGES ON SCHEMA test to bob;
SET ROLE bob;
SELECT * FROM test.test_table1; -- this fail !
```

Bob can now do everything on the schema. But it does not mean that you can do everything on child elements of the schema (tables, sequences, etc...). You need to explicitly grant privileges to all tables to bob:

```SQL
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA test TO bob;
SET ROLE bob;
SELECT * FROM test.test_table1;
```


## Grant and default

Worst than that. There are privileges and default privileges ! see for yourself :

```SQL
SET ROLE postgres; -- or superuser
CREATE TABLE test.test_table2 (id uuid, data2 int NOT NULL);
SET ROLE bob;
SELECT * FROM test.test_table2; -- FAIL
```

In order to this to work, you need to add default privileges :

```SQL
SET ROLE postgres;
ALTER DEFAULT PRIVILEGES FOR ROLE postgres IN SCHEMA test GRANT ALL PRIVILEGES ON ALL TABLES to bob; 
```

This lines contains quite a lot ! You are modifying default privileges (ie: privileges that will be applied on newly created tables/sequences/etc...). Default privileges created by a specific roles (postgres), in a specific schema (test), then on top of that you gives a normal grant. The part `FOR ROLE postgres` takes me quite a bit to understand.

## Grant and default: what the heck ?

If you have multiple users that create tables, you will need to put this role name in the `FOR ROLE` clause. If you don't, you'll run into privileges errors.

 
## Backup user

There are a lot of strategy to backup (and restore !) your db. The most simple one is `pg_dump` (and `pg_dumpall`). The backup user need to have access to all table of all schema. I am creating a user with superuser privileges, but with only read capabilities :

```SQL
CREATE ROLE backup WITH SUPERUSER LOGIN ENCRYPTED PASSWORD 'a super strong password' INHERIT;
ALTER USER backup SET default_transaction_read_only = ON;
```

I found this super usefull !
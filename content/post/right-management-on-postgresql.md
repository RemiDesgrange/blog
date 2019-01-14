---
title: "Right Management in Postgresql"
date: 2017-11-28T10:29:57+01:00
daft: true
---

Recently I though about (re)starting a blog, but didn't find subject to fill it. Then I got a new job with tons of news subject to learn and I need a place to store it, maybe it will help some. Enjoy.

# Right Management in Postgresql

[@Fibrea](https://twitter.com/Fibrea73) we use postgresql a lot, it's the heart of our system. Operating this DB  day to day is quite challenging some time. So here is what I understand from my experience and reading on right management in postgresql.


# Roles, User, Group

On postgresql there is only roles, a user is a role, and a group is role. `CREATE USER x ` is equal to `CREATE ROLE x LOGIN`. You can create two users (so two roles), let's say `bob` and `alice`, and then do `GRANT alice to bob`. There is also inheritence, but the [manual is explaining it well](https://www.postgresql.org/docs/9.4/static/sql-createrole.html)

# Giving Grant to what

On postgresql there are strange behavior (at first) if you want to control what happend in your DB. You can totaly grant all privileges to a super admin and using it everywhere, but for real life case that's usualy not the best option.


## Grant and default

Imagine the user john created earlier. You can grant him all privileges on a table (or schema) :

```SQL
GRANT ALL PRIVILEGES ON TABLE test TO john;
```
 

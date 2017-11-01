---
featured_image: "/img/mackellar-american-printer-1882-pivotal-caster-1200rgb-2048x.jpg"
title: "How to get a list of MySQL user accounts?"
description: "First steps using psql and catalog queries"
---

> When using `psql`, use the `\du` command.

Here's a sample output of running that command when using the
[psql](https://www.postgresql.org/docs/current/static/app-psql.html) command
line tool:

~~~
~# \du
                                   List of roles
 Role name │                         Attributes                         │ Member of 
═══════════╪════════════════════════════════════════════════════════════╪═══════════
 app       │                                                            │ {}
 appdev    │ Create role                                                │ {}
 cdstore   │                                                            │ {appdev}
 critical  │                                                            │ {app}
 dbowner   │                                                            │ {}
 dim       │ Superuser, Create role, Create DB                          │ {}
 dontcare  │                                                            │ {app}
 f1db      │                                                            │ {appdev}
 notsomuch │                                                            │ {app}
 postgres  │ Superuser, Create role, Create DB, Replication, Bypass RLS │ {}
~~~

The `psql` application supports *readline* history and completion. It's a
complete PostgreSQL client that I recommend trying out. One of its feature
is allowing to see every catalog queries the client is doing, with the
`ECHO_HIDDEN` setting:

~~~
~# \set ECHO_HIDDEN on
~# \du
********* QUERY **********
SELECT r.rolname, r.rolsuper, r.rolinherit,
  r.rolcreaterole, r.rolcreatedb, r.rolcanlogin,
  r.rolconnlimit, r.rolvaliduntil,
  ARRAY(SELECT b.rolname
        FROM pg_catalog.pg_auth_members m
        JOIN pg_catalog.pg_roles b ON (m.roleid = b.oid)
        WHERE m.member = r.oid) as memberof
, r.rolreplication
, r.rolbypassrls
FROM pg_catalog.pg_roles r
WHERE r.rolname !~ '^pg_'
ORDER BY 1;
**************************
~~~

So `psql` is querying the `pg_catalog.pg_roles` catalog table to fetch
information, and then doing some efforts to have the data displayed in a
human-readable format. Maybe your application doesn't need so much efforts,
and the following query is all you need:

~~~
select rolname
  from pg_catalog.pg_roles
 where rolcanlogin;
~~~

And here's the list of local roles that are allowed to login:

~~~
  rolname  
═══════════
 postgres
 dim
 dbowner
 app
 critical
 notsomuch
 dontcare
 appdev
 cdstore
 f1db
(10 rows)
~~~

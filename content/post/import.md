---
featured_image: "/img/mackellar-american-printer-1882-pivotal-caster-1200rgb-2048x.jpg"
title: "How to import an SQL file using the command line in MySQL?"
description: "The psql include facility, and the COPY command and protocol."
---

> Use the `\i` command in `psql`, or the command line `psql -a -f file.sql`

The PostgreSQL
[psql](https://www.postgresql.org/docs/current/static/app-psql.html) command
line tool knows how to parse a file containing SQL statements and send them
all to a PostgreSQL server. Several ways are possible.

If you have a `magic.create.sql` file that contains the following:

~~~
create schema if not exists magic;
create table magic.allsets(data jsonb);
~~~

It is possible to connect to your target database then have `psql` execute
the commands from the file in the following way:

~~~
$ psql foo
foo# \i magic.create.sql
CREATE SCHEMA
CREATE TABLE
~~~

The following alternative shows both the queries and their result:

~~~
$ psql -d foo -a -f magic.create.sql
create schema if not exists magic;
CREATE SCHEMA
create table magic.allsets(data jsonb);
CREATE TABLE
~~~

In case your SQL script doesn't contain an explicit transaction, it's also
possible to have `psql` wrap your commands in a transaction for you, using
the `--single-transaction` switch.

Also, you might want to use a `~/.psqlrc` setup such as the following, where
we have `ON_ERROR_STOP` set to `on`, so that, you know, we stop at the first
error.

~~~
\set PROMPT1 '%~%x%# '
\x auto
\set ON_ERROR_STOP on
\set ON_ERROR_ROLLBACK interactive

-- \set VERBOSITY verbose

\pset null '⦱'
\pset null '¤'
\pset linestyle 'unicode'

\pset unicode_border_linestyle single
\pset unicode_column_linestyle single
\pset unicode_header_linestyle double

set intervalstyle to 'postgres_verbose';

\setenv LESS '-iMFXSx4R'
\setenv EDITOR '/Applications/Emacs.app/Contents/MacOS/bin/emacsclient -nw'
~~~

When using that setup, pay attention the the `EDITOR` setting and replace it
with your own local preference. Then play with `\e` in `psql`.

Now of course the output really is verbose:

~~~
~ psql -d foo -a --single-transaction -f magic.create.sql
\set PROMPT1 '%~%x%# '
\x auto
Expanded display is used automatically.
\set ON_ERROR_STOP on
\set ON_ERROR_ROLLBACK interactive
-- \set VERBOSITY verbose
\pset null '⦱'
Null display is "⦱".
\pset null '¤'
Null display is "¤".
\pset linestyle 'unicode'
Line style is unicode.
\pset unicode_border_linestyle single
Unicode border line style is "single".
\pset unicode_column_linestyle single
Unicode column line style is "single".
\pset unicode_header_linestyle double
Unicode header line style is "double".
set intervalstyle to 'postgres_verbose';
SET
\setenv LESS '-iMFXSx4R'
\setenv EDITOR '/Applications/Emacs.app/Contents/MacOS/bin/emacsclient -nw'
create schema if not exists magic;
CREATE SCHEMA
create table magic.allsets(data jsonb);
CREATE TABLE
~~~


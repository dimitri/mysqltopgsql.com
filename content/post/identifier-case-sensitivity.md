---
icon: "fa-solid fa-font"
title: "Why did my queries break after migrating — is PostgreSQL case-sensitive?"
description: "myTable and mytable are the same table in MySQL, and two different things in PostgreSQL. The most common silent breakage after cutover."
weight: 4
---

> Unquoted identifiers in PostgreSQL are folded to lowercase — always,
> everywhere, on every platform. MySQL's answer to "is `MyTable` the same
> as `mytable`?" is `it depends` — on the storage engine, and on the
> filesystem underneath it. This is one of the issues most likely to
> break silently after a migration, because nothing errors until a
> quoted reference stops matching.

# Why MySQL's answer is "it depends"

MySQL's historical storage engines represent each table as one or more
files on disk — MyISAM does this directly (a `.MYD`/`.MYI` pair per
table), and InnoDB does too when `innodb_file_per_table` is enabled
(the modern default). That means table-name case-sensitivity in MySQL
isn't really a MySQL decision at all — it's inherited from whatever the
underlying filesystem does with file names:

- On Linux (ext4 and friends, case-sensitive), table names are
  case-sensitive by default — `MyTable` and `mytable` are two different
  files, hence two different tables.
- On **macOS**, the default filesystem (APFS or HFS+) is
  case-insensitive but case-*preserving* — `MyTable.ibd` and
  `mytable.ibd` collide, so MySQL folds table names to a consistent case
  to avoid that collision.
- On Windows, same story as macOS — case-insensitive filesystem, same
  workaround.

MySQL exposes this directly as the `lower_case_table_names` system
variable (0 = case-sensitive, matching Linux; 1 or 2 = fold or
preserve-but-compare-insensitively, matching macOS/Windows) — and
because it's read from the filesystem's own behavior at data-directory
creation time, the same MySQL schema can behave differently depending on
which machine it was first set up on. Column names and index names don't
have this problem — they're not one-file-per-object the way tables are —
which is exactly why this gotcha is specifically about table (and
database) names, not every identifier in your schema.

# PostgreSQL has none of this

PostgreSQL doesn't map tables onto case-sensitive OS files the way
MySQL's engines historically have — table data lives inside PostgreSQL's
own storage files, addressed by internal object IDs, not by a filename
derived from the table name. So there's no filesystem to inherit
case-sensitivity quirks from, no `lower_case_table_names`-equivalent
variable to configure, and no difference in behavior between Linux,
macOS, or Windows. The rule is exactly one rule, everywhere: unquoted
folds to lowercase, quoted is used exactly as written.

# Where this actually bites

Two different codepaths break for two different reasons:

- **Hand-written SQL that quotes identifiers.** `SELECT * FROM "MyTable"`
  in PostgreSQL looks for a table literally named `MyTable` — case
  preserved because it's quoted — and fails if the table was actually
  created as `mytable` (the default, since `CREATE TABLE MyTable (...)`
  with no quotes gets folded to lowercase on creation too).
- **ORMs and migration tools that generate mixed-case names.** Frameworks
  that model class names as table names can generate DDL or queries that
  assume MySQL's more permissive matching, and start failing lookups the
  moment PostgreSQL's lowercase-folding rule applies.

# The rule, precisely

PostgreSQL's actual behavior (from the
[SQL Syntax: Identifiers](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-IDENTIFIERS)
documentation): an unquoted identifier is folded to lowercase before
anything else happens. A double-quoted identifier is used exactly as
written, case included, and matched exactly as written from then on. So:

~~~ sql
CREATE TABLE MyTable (id int);   -- creates: mytable
SELECT * FROM MyTable;           -- works: folds to mytable, matches
SELECT * FROM "MyTable";         -- fails: no table named MyTable exists
SELECT * FROM mytable;           -- works: folds to mytable, matches
~~~

# What to do about it

Pick one convention and be consistent, rather than fighting the fold:

- **Simplest**: use lowercase, unquoted identifiers everywhere in new
  PostgreSQL schema and queries — snake_case table and column names, no
  quoting required, no surprises.
- **If your ORM insists on mixed case**: check whether it quotes
  identifiers consistently on every generated query, not just on
  `CREATE TABLE`. Inconsistency between the two is exactly what produces
  the "works in dev, breaks after deploy" version of this bug.

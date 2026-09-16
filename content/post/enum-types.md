---
icon: "fa-solid fa-list-check"
title: "How does MySQL's ENUM type migrate to PostgreSQL?"
description: "MySQL declares ENUM inline on the column; PostgreSQL needs a separate CREATE TYPE. pgloader does the translation automatically."
weight: 6
---

> MySQL declares an `ENUM` inline, right on the column definition.
> PostgreSQL has no inline enum — you `CREATE TYPE` a named enum type
> first, then use it as a column type. pgloader bridges this automatically,
> with no casting rule required for the default case.

# What pgloader's own docs say

Straight from pgloader's MySQL reference documentation:

> Enum types are declared inline in MySQL and separately with a `CREATE
> TYPE` command in PostgreSQL, so each column of Enum Type is converted to
> a type named after the table and column names defined with the same
> labels in the same order.

In other words: for a MySQL column like

~~~ sql
CREATE TABLE orders (
  id     int PRIMARY KEY,
  status ENUM('pending', 'shipped', 'delivered')
);
~~~

pgloader automatically issues the PostgreSQL equivalent, naming the new
type after the table and column it came from:

~~~ sql
CREATE TYPE orders_status AS ENUM ('pending', 'shipped', 'delivered');

CREATE TABLE orders (
  id     int PRIMARY KEY,
  status orders_status
);
~~~

Same labels, same order, no manual `CREATE TYPE` statement to write by
hand, and no casting rule needed in your `.load` file for the common case
— this is pgloader's default behavior, not something you opt into.

# When you'd still write a rule

If you want a specific type name instead of the auto-generated
`tablename_columnname` pattern, or you're consolidating the same set of
labels used across several MySQL columns into one shared PostgreSQL enum
type, that's where a custom `CAST` rule comes in — the same casting-rules
mechanism covered in the [Casting Rules &amp; Schema Mapping module](/course/#module-6)
of the full course. For the default case, though, there's nothing to
configure.

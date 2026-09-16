---
icon: "fa-solid fa-calendar-xmark"
title: "How do I handle MySQL's 0000-00-00 zero dates in PostgreSQL?"
description: "MySQL allows an all-zero date as a value; PostgreSQL doesn't. Here's the pgloader transform that fixes it during migration."
weight: 5
---

> MySQL allows `0000-00-00` (and `0000-00-00 00:00:00`) as a literal date
> value — a real, storable value, not an error. PostgreSQL has no such
> thing: there is no year zero, so `date`/`timestamp` columns simply reject
> it. pgloader has a purpose-built transform for exactly this,
> `zero-dates-to-null`.

This is one of the sharper migration gotchas — it doesn't show up as a
loud error message you can grep for, it shows up as rows that silently
fail to load, or as a `NULL` where you didn't expect one, unless you tell
pgloader what to do with it up front.

# The transform, straight from pgloader's own docs

pgloader ships a transformation function built for this exact case,
documented in its reference manual:

> **zero-dates-to-null**
>
> When the input date is all zeroes, return `nil`, which gets loaded as a
> PostgreSQL `NULL` value.

# Applying it in a `.load` command

Here's the real casting rule from pgloader's own MySQL migration reference,
applied to every date/datetime/timestamp column with a zero-date default:

~~~
CAST type datetime when default "0000-00-00 00:00:00" and not null
    to timestamptz drop not null drop default
    using zero-dates-to-null,

     type datetime when default "0000-00-00 00:00:00"
    to timestamptz drop default
    using zero-dates-to-null,

     type timestamp when default "0000-00-00 00:00:00" and not null
    to timestamptz drop not null drop default
    using zero-dates-to-null,

     type date when default "0000-00-00"
    to date drop default
    using zero-dates-to-null;
~~~

Each rule matches a MySQL column definition by its exact zero-date
default, casts it to the right PostgreSQL type, drops the now-meaningless
default, and runs every value through `zero-dates-to-null` on the way in —
so a zero date becomes a real `NULL` instead of a load failure.

# Why this needs a rule at all

PostgreSQL's date/timestamp types are strict by design — there's no
implicit "close enough" value the way `0000-00-00` behaves in MySQL. If
your schema uses zero dates as a stand-in for "no date yet" (a common
pattern for optional fields with a `NOT NULL` constraint MySQL was happy
to satisfy with zeroes), decide up front whether the PostgreSQL column
should actually be nullable — `drop not null` in the rules above says yes,
it should.

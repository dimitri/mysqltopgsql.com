---
icon: "fa-solid fa-hashtag"
title: "What's the PostgreSQL equivalent of MySQL's LAST_INSERT_ID()?"
description: "There's no LAST_INSERT_ID() function in PostgreSQL — use RETURNING to get the new id back from the same INSERT statement."
weight: 7
---

> There's no direct equivalent function. Instead, PostgreSQL lets you get
> the generated id back from the `INSERT` itself, with a `RETURNING`
> clause — one round trip, no separate lookup call needed.

# The MySQL pattern

~~~ sql
INSERT INTO orders (customer_id) VALUES (42);
SELECT LAST_INSERT_ID();
~~~

Two statements, and — worth knowing even before you migrate —
`LAST_INSERT_ID()` is connection-scoped, so it's already fragile around
connection pooling if you're not careful about which connection issued the
insert.

# The PostgreSQL way: RETURNING

~~~ sql
INSERT INTO orders (customer_id) VALUES (42)
RETURNING id;
~~~

One statement, one round trip, and the returned id is unambiguously tied
to the row you just inserted — no connection-scoped state to reason
about. `RETURNING` isn't limited to `id`, either: return any column, or
several, or `RETURNING *` for the whole row.

# If you're not doing the INSERT yourself

Working through a driver or ORM instead of raw SQL, look for whichever
call actually executes an `INSERT ... RETURNING` under the hood — most
PostgreSQL drivers and ORMs use this pattern already for "give me back the
row I just created," you're just not writing the SQL by hand. If you're
stuck without `RETURNING` for some reason, `currval('sequence_name')` is
the direct sequence-based equivalent — see the
[AUTO_INCREMENT / sequences FAQ](/post/sequences/) for how PostgreSQL
sequences map to MySQL's AUTO_INCREMENT in the first place.

---
featured_image: "/img/mackellar-american-printer-1882-pivotal-caster-1200rgb-2048x.jpg"
title: "Which MySQL data type to use for storing boolean values?"
description: "PostgreSQL has a boolean data type for you"
---

> In PostgreSQL, use the
> [boolean](https://www.postgresql.org/docs/current/static/datatype-boolean.html)
> data type. Done.

When using [pgloader](http://pgloader.io) to migrate your MySQL database to
PostgreSQL, the following cast rule is the default:

~~~
CAST type tinyint to boolean using tinyint-to-boolean
~~~

It's of course then possible to migrate a `tinyint` column from MySQL to a
PostgreSQL `smallint` with the following specific rule:

~~~
CAST column table_a.count to smallint
~~~

# Boolean Columns

When you have a `boolean` column in PostgreSQL then things work as you would
expect. You can query rows being true:

~~~
   select *
     from table
    where col is true;
~~~

# Three-Valued Logic

Be careful with boolean logic in the standard SQL though, as the standard
implements Three-Valued Logic, as seen here:

~~~
select a::text, b::text,
       (a=b)::text as "a=b",
       format('%s = %s',
              coalesce(a::text, 'null'),
              coalesce(b::text, 'null')) as op,
       format('is %s',
              coalesce((a=b)::text, 'null')) as result
  from (values(true), (false), (null)) v1(a)
       cross join
       (values(true), (false), (null)) v2(b);
~~~

Here's our SQL boolean truth table:

~~~
   a   │   b   │  a=b  │      op       │  result  
═══════╪═══════╪═══════╪═══════════════╪══════════
 true  │ true  │ true  │ true = true   │ is true
 true  │ false │ false │ true = false  │ is false
 true  │ ¤     │ ¤     │ true = null   │ is null
 false │ true  │ false │ false = true  │ is false
 false │ false │ true  │ false = false │ is true
 false │ ¤     │ ¤     │ false = null  │ is null
 ¤     │ true  │ ¤     │ null = true   │ is null
 ¤     │ false │ ¤     │ null = false  │ is null
 ¤     │ ¤     │ ¤     │ null = null   │ is null
(9 rows)
~~~

If you want to avoid the trap of `null = null` returning NULL, then you can
use the standard operators IS DISTINCT FROM and IS NOT DISTINCT FROM.

~~~ sql
select a::text as left, b::text as right,
       (a = b)::text as "=",
       (a <> b)::text as "<>",
       (a is distinct from b)::text as "is distinct",
       (a is not distinct from b)::text as "is not distinct from"
  from            (values(true),(false),(null)) t1(a)
       cross join (values(true),(false),(null)) t2(b);
~~~

And here's the table this time:

~~~ psql
 left  │ right │   =   │  <>   │ is distinct │ is not distinct from 
═══════╪═══════╪═══════╪═══════╪═════════════╪══════════════════════
 true  │ true  │ true  │ false │ false       │ true
 true  │ false │ false │ true  │ true        │ false
 true  │ ¤     │ ¤     │ ¤     │ true        │ false
 false │ true  │ false │ true  │ true        │ false
 false │ false │ true  │ false │ false       │ true
 false │ ¤     │ ¤     │ ¤     │ true        │ false
 ¤     │ true  │ ¤     │ ¤     │ true        │ false
 ¤     │ false │ ¤     │ ¤     │ true        │ false
 ¤     │ ¤     │ ¤     │ ¤     │ false       │ true
(9 rows)
~~~

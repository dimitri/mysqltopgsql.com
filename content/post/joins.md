---
featured_image: "/img/mackellar-american-printer-1882-pivotal-caster-1200rgb-2048x.jpg"
title: "What's the difference between INNER JOIN, LEFT JOIN, RIGHT JOIN and FULL JOIN?"
description: "SQL basics explained"
---

> A JOIN is an operation that applies to two relations and returns a
> relation. The different kinds of JOINs implement different operations. The
> PostgreSQL documentation section [Table
> Expressions](https://www.postgresql.org/docs/current/static/queries-table-expressions.html)
> gives all the details.

In the same way that `+` and `-` and `*` are different operations in maths,
the different kind of joins are different relational operations.

The main thing is to remember that we are working with relations, and a JOIN
applies to a pair of relations at a time.

The INNER and OUTER keywords in SQL are *noise* words. You don't have to use
them, as they bear no semantics. A JOIN is always an INNER JOIN, a LEFT JOIN
is always a LEFT OUTER JOIN.

We're going to use the <http://ergast.com/mrd/db/> database for our
examples. You can download the MySQL dump of this database, then once
restored into your MySQL instance, load it to PostgreSQL with
[pgloader](http://pgloader.io):

~~~
$ createdb f1db
$ pgloader mysql://root@localhost/f1db pgsql:///f1db
~~~

# *Inner* Join

An Inner Join is an intersection operation. From the PostgreSQL docs:

> For each row R1 of T1, the joined table has a row for each row in T2 that
> satisfies the join condition with R1.

In the f1db database, we can associate seasons and races so as to display
for each race the Wikipedia URL of the season:

~~~
  select name, date, seasons.url
    from      seasons
         join races using(year)
order by date desc
   limit 3;
~~~

And here's our result set:

~~~
         name         │    date    │                          url                          
══════════════════════╪════════════╪═══════════════════════════════════════════════════════
 Abu Dhabi Grand Prix │ 2017-11-26 │ https://en.wikipedia.org/wiki/2017_Formula_One_season
 Brazilian Grand Prix │ 2017-11-12 │ https://en.wikipedia.org/wiki/2017_Formula_One_season
 Mexican Grand Prix   │ 2017-10-29 │ https://en.wikipedia.org/wiki/2017_Formula_One_season
(3 rows)
~~~


Another way to write the same query is:

~~~
  select name, date, seasons.url
    from      seasons
         join races
           on seasons.year = races.year
order by date desc
   limit 3;
~~~

We replaced the `using()` syntax with a more generic `on` clause for the
join condition. Of course the result is exactly the same as before.

# Left *Outer* Join

> First, an inner join is performed. Then, for each row in T1 that does not
> satisfy the join condition with any row in T2, a joined row is added with
> null values in columns of T2. Thus, the joined table always has at least
> one row for each row in T1.

The f1db database has every lap and pitstop recorded. Of course not every
driver makes it to the pitstop at every lap, so if we want to display
information about both laps ahd pitstops within the same relation, we need a
LEFT JOIN as in the following query:

~~~
   select drivers.surname,
          lap,
          laptimes.milliseconds * interval '1 ms' as laptime,
          position,
          stop,
          pitstops.milliseconds * interval '1ms' as stoptime

    from           laptimes
         left join pitstops using(raceid, driverid, lap)
         join drivers using(driverid)

   where raceid=972
     and position between 1 and 3
     and lap between 28 and 35

order by lap, position;
~~~

We only look at the first three drivers of the race, and in order not to
have too many rows to show we limit the result set to where pitstops
happened, which apparently was in between the 28th and 35th laps.

Thanks to using a LEFT JOIN, we get NULL values for the `stop` and
`stoptime` columns for laps where our selected drivers didn't show up at the
pit stop.

~~~
  surname  │ lap │       laptime       │ position │ stop │   stoptime    
═══════════╪═════╪═════════════════════╪══════════╪══════╪═══════════════
 Vettel    │  28 │ @ 1 min 39.192 secs │        1 │    ¤ │ ¤
 Räikkönen │  28 │ @ 1 min 39.486 secs │        2 │    ¤ │ ¤
 Hamilton  │  28 │ @ 1 min 38.913 secs │        3 │    ¤ │ ¤
 Vettel    │  29 │ @ 1 min 38.828 secs │        1 │    ¤ │ ¤
 Räikkönen │  29 │ @ 1 min 43.57 secs  │        2 │    1 │ @ 30.152 secs
 Hamilton  │  29 │ @ 1 min 39.685 secs │        3 │    ¤ │ ¤
 Vettel    │  30 │ @ 1 min 38.291 secs │        1 │    ¤ │ ¤
 Hamilton  │  30 │ @ 1 min 44.91 secs  │        2 │    1 │ @ 29.739 secs
 Bottas    │  30 │ @ 1 min 38.208 secs │        3 │    ¤ │ ¤
 Vettel    │  31 │ @ 1 min 38.347 secs │        1 │    ¤ │ ¤
 Bottas    │  31 │ @ 1 min 38.16 secs  │        2 │    ¤ │ ¤
 Räikkönen │  31 │ @ 1 min 37.372 secs │        3 │    ¤ │ ¤
 Vettel    │  32 │ @ 1 min 38.306 secs │        1 │    ¤ │ ¤
 Bottas    │  32 │ @ 1 min 38.704 secs │        2 │    ¤ │ ¤
 Räikkönen │  32 │ @ 1 min 37.832 secs │        3 │    ¤ │ ¤
 Vettel    │  33 │ @ 1 min 38.553 secs │        1 │    ¤ │ ¤
 Bottas    │  33 │ @ 1 min 39.169 secs │        2 │    ¤ │ ¤
 Räikkönen │  33 │ @ 1 min 37.958 secs │        3 │    ¤ │ ¤
 Vettel    │  34 │ @ 1 min 43.112 secs │        1 │    1 │ @ 30.097 secs
 Bottas    │  34 │ @ 1 min 38.426 secs │        2 │    ¤ │ ¤
 Räikkönen │  34 │ @ 1 min 38.091 secs │        3 │    ¤ │ ¤
 Bottas    │  35 │ @ 1 min 37.978 secs │        1 │    ¤ │ ¤
 Vettel    │  35 │ @ 1 min 58.663 secs │        2 │    ¤ │ ¤
 Räikkönen │  35 │ @ 1 min 38.715 secs │        3 │    ¤ │ ¤
(24 rows)
~~~

# Right *Outer* Join

> First, an inner join is performed. Then, for each row in T2 that does not
> satisfy the join condition with any row in T1, a joined row is added with
> null values in columns of T1. This is the converse of a left join: the
> result table will always have a row for each row in T2.

So basically the previous query can be written as a RIGHT JOIN if we
consider the *pitstops* first:

~~~
   select drivers.surname,
          lap,
          laptimes.milliseconds * interval '1 ms' as laptime,
          position,
          stop,
          pitstops.milliseconds * interval '1ms' as stoptime

    from            pitstops
         right join laptimes using(raceid, driverid, lap)
         join drivers using(driverid)

   where raceid=972
     and position between 1 and 3
     and lap between 28 and 35

order by lap, position;
~~~

We get the same results as before, of course.

# Lateral Join

With PostgreSQL it's also possible to use a LATERAL JOIN. This allows to
implement a parameterized subquery for every row found in a table.

> When a FROM item contains LATERAL cross-references, evaluation proceeds as
> follows: for each row of the FROM item providing the cross-referenced
> column(s), or set of rows of multiple FROM items providing the columns,
> the LATERAL item is evaluated using that row or row set's values of the
> columns. The resulting row(s) are joined as usual with the rows they were
> computed from. This is repeated for each row or set of rows from the
> column source table(s).

A good example of a LEFT JOIN LATERAL is a top-n query. In the following
example we find the top three drivers per seasons, by how many points they
accumulated each season:

~~~
select seasons.year,
       drivers.code,
       format('%s %s', drivers.forename, drivers.surname) as driver,
       points
   from seasons
        left join lateral
        (
           select races.year, driverid, sum(points) as points
             from results join races using(raceid)
            where races.year = seasons.year
         group by year, driverid
         order by points desc
            limit 3
         )
         top3 on true
         join drivers using(driverid)
    where seasons.year between 2012 and 2016
order by seasons.year, points desc;
~~~

One thing to note about the LEFT JOIN LATERAL is that the join condition is
implemented in the subquery's WHERE clause. Then we issue the strange `ON
TRUE` join condition so that we can respect the standard SQL syntax for join
operators.

We arbitrary limit the number of rows in the output by restricting the
result set to seasons from 2012 to 2016, and here's our result:

~~~
 year │ code │      driver      │ points 
══════╪══════╪══════════════════╪════════
 2012 │ VET  │ Sebastian Vettel │    281
 2012 │ ALO  │ Fernando Alonso  │    278
 2012 │ RAI  │ Kimi Räikkönen   │    207
 2013 │ VET  │ Sebastian Vettel │    397
 2013 │ ALO  │ Fernando Alonso  │    242
 2013 │ WEB  │ Mark Webber      │    199
 2014 │ HAM  │ Lewis Hamilton   │    384
 2014 │ ROS  │ Nico Rosberg     │    317
 2014 │ RIC  │ Daniel Ricciardo │    238
 2015 │ HAM  │ Lewis Hamilton   │    381
 2015 │ ROS  │ Nico Rosberg     │    322
 2015 │ VET  │ Sebastian Vettel │    278
 2016 │ ROS  │ Nico Rosberg     │    385
 2016 │ HAM  │ Lewis Hamilton   │    380
 2016 │ RIC  │ Daniel Ricciardo │    256
(15 rows)
~~~

# Full *Outer* Join

> First, an inner join is performed. Then, for each row in T1 that does not
> satisfy the join condition with any row in T2, a joined row is added with
> null values in columns of T2. Also, for each row of T2 that does not
> satisfy the join condition with any row in T1, a joined row with null
> values in the columns of T1 is added.

TODO: find a good FULL OUTER JOIN example in the f1db model.

# Cross Join

> For every possible combination of rows from T1 and T2 (i.e., a Cartesian
> product), the joined table will contain a row consisting of all columns in
> T1 followed by all columns in T2. If the tables have N and M rows
> respectively, the joined table will have N * M rows.
>
> FROM T1 CROSS JOIN T2 is equivalent to FROM T1 INNER JOIN T2 ON TRUE (see
> below). It is also equivalent to FROM T1, T2.

TODO: find a good CROSS JOIN example in the f1db model.

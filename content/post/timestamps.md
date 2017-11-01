---
featured_image: "/img/mackellar-american-printer-1882-pivotal-caster-1200rgb-2048x.jpg"
title: "Should I use field 'datetime' or 'timestamp'?"
description: "And what about the time zone, too?"
---

> Always use `timestamp with time zone`, or its spelling `timestamptz`, when
> using PostgreSQL.

The calendar is something very complex, and easy to get wrong. Do you know
about ISO week numbers, leap years, and time zone political changes? How do
compute which day is tomorrow?

Did you know that the year before [AD 1](https://en.wikipedia.org/wiki/AD_1)
in our Gregorian calendar is the year [1
BC](https://en.wikipedia.org/wiki/1_BC)? Yes, right, there's no year zero in
our calendar.

# The Calendar and PostgreSQL

PostgreSQL is very good at the calendar. Let's see a show-off query about
that, with some computations that you might need to implement in your
application's code:

~~~ sql
select date::date,
       extract('isodow' from date) as dow,
       to_char(date, 'dy') as day,
       extract('isoyear' from date) as "iso year",
       extract('week' from date) as week,
       extract('day' from
               (date + interval '2 month - 1 day')
              )
        as feb,
       extract('year' from date) as year,
       extract('day' from
               (date + interval '2 month - 1 day')
              ) = 29
       as leap
  from generate_series(date '2000-01-01',
                       date '2010-01-01',
                       interval '1 year')
       as t(date);
~~~

The *generate_series* function returns a set of items, here all the dates of
the first day of the years from the 2000's decade. For each of them we then
compute several calendar based values:

~~~
    date    │ dow │ day │ iso year │ week │ feb │ year │ leap 
════════════╪═════╪═════╪══════════╪══════╪═════╪══════╪══════
 2000-01-01 │   6 │ sat │     1999 │   52 │  29 │ 2000 │ t
 2001-01-01 │   1 │ mon │     2001 │    1 │  28 │ 2001 │ f
 2002-01-01 │   2 │ tue │     2002 │    1 │  28 │ 2002 │ f
 2003-01-01 │   3 │ wed │     2003 │    1 │  28 │ 2003 │ f
 2004-01-01 │   4 │ thu │     2004 │    1 │  29 │ 2004 │ t
 2005-01-01 │   6 │ sat │     2004 │   53 │  28 │ 2005 │ f
 2006-01-01 │   7 │ sun │     2005 │   52 │  28 │ 2006 │ f
 2007-01-01 │   1 │ mon │     2007 │    1 │  28 │ 2007 │ f
 2008-01-01 │   2 │ tue │     2008 │    1 │  29 │ 2008 │ t
 2009-01-01 │   4 │ thu │     2009 │    1 │  28 │ 2009 │ f
 2010-01-01 │   5 │ fri │     2009 │   53 │  28 │ 2010 │ f
(11 rows)
~~~

The *iso year* column looks strange, with its value being 1999 for the first
of January, 2000. Well the rules are the following:

> _By definition, ISO weeks start on Mondays and the first week of a year
> contains January 4 of that year. In other words, the first Thursday of a
> year is in week 1 of that year._

# PostgreSQL date queries

A typical PostgreSQL query using a date range restriction looks like the
following:

~~~
select date, name, drivers.surname as winner
  from races
       left join results
              on results.raceid = races.raceid
             and results.position = 1
       left join drivers using(driverid)
 where date >= date '2017-04-01'
   and date <  date '2017-04-01' + 3 * interval '1 month';
~~~

Such a spelling allows PostgreSQL to use an index on the `date` column, if
there's one defined. Other spellings using `extract()` to compare separately
the year, month and days actively prevent using an index. Avoid them.

PostgreSQL implements *decorated literals*. Those are literal values given
in the SQL query, with a data type hint so that PostgreSQL doesn't have to
guess what this literal represent. In the previous query we use the `date
'2017-04-01'` syntax, so that PostgreSQL parses the `'2017-04-01'` value as
a *date* representation.

PostgreSQL implements the *interval* data type, and knows how to compute how
many days are in a *month* when the interval value is used in relation with
a date, as in our example query above.

And if you're curious here's the result:

~~~
    date    │         name          │  winner  
════════════╪═══════════════════════╪══════════
 2017-04-09 │ Chinese Grand Prix    │ Hamilton
 2017-04-16 │ Bahrain Grand Prix    │ Vettel
 2017-04-30 │ Russian Grand Prix    │ Bottas
 2017-05-14 │ Spanish Grand Prix    │ Hamilton
 2017-05-28 │ Monaco Grand Prix     │ Vettel
 2017-06-11 │ Canadian Grand Prix   │ Hamilton
 2017-06-25 │ Azerbaijan Grand Prix │ ¤
(7 rows)
~~~


---
featured_image: "/img/mackellar-american-printer-1882-pivotal-caster-1200rgb-2048x.jpg"
title: "How to output MySQL query results in CSV format?"
description: "PostgreSQL COPY and psql output formats."
---

> PostgreSQL implements the COPY protocol and command to output data
> formatted in CSV. The `psql` command line client also knows how to produce
> some other formats, such as XML, asciidoc, or latex.

The simplest way to have your query result in CSV is the following:

~~~
copy (
      select format('%s %s', drivers.forename, drivers.surname) as name,
             sum(points) as points
        from drivers
             join results using(driverid)
             join races using(raceid)
       where races.year = 2016
    group by drivers.driverid
    order by points desc
)
to stdout with csv header;
~~~

Which gives:

~~~
name,points
Nico Rosberg,385
Lewis Hamilton,380
Daniel Ricciardo,256
Sebastian Vettel,212
Max Verstappen,204
Kimi Räikkönen,186
Sergio Pérez,101
Valtteri Bottas,85
Nico Hülkenberg,72
Fernando Alonso,54
Felipe Massa,53
Carlos Sainz,46
Romain Grosjean,29
Daniil Kvyat,25
Jenson Button,21
Kevin Magnussen,7
Felipe Nasr,2
Pascal Wehrlein,1
Jolyon Palmer,1
Stoffel Vandoorne,1
Esteban Gutiérrez,0
Marcus Ericsson,0
Esteban Ocon,0
Rio Haryanto,0
~~~

It is possible to change the delimiter of course:

~~~
copy (
      select format('%s %s', drivers.forename, drivers.surname) as name,
             sum(points) as points
        from drivers
             join results using(driverid)
             join races using(raceid)
       where races.year = 2016
    group by drivers.driverid
    order by points desc
)
to stdout with csv header delimiter E'\t';
~~~

This time we have:

~~~
name	points
Nico Rosberg	385
Lewis Hamilton	380
Daniel Ricciardo	256
Sebastian Vettel	212
Max Verstappen	204
Kimi Räikkönen	186
Sergio Pérez	101
Valtteri Bottas	85
Nico Hülkenberg	72
Fernando Alonso	54
Felipe Massa	53
Carlos Sainz	46
Romain Grosjean	29
Daniil Kvyat	25
Jenson Button	21
Kevin Magnussen	7
Felipe Nasr	2
Pascal Wehrlein	1
Jolyon Palmer	1
Stoffel Vandoorne	1
Esteban Gutiérrez	0
Marcus Ericsson	0
Esteban Ocon	0
Rio Haryanto	0
~~~

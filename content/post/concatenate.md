---
featured_image: "/img/mackellar-american-printer-1882-pivotal-caster-1200rgb-2048x.jpg"
title: "Can I concatenate multiple MySQL rows into one field?"
description: "Yes sure, that's called an AGGREGATE."
---

> Concatenating values from several rows as a single value summarizing all
> of them is done by using an AGGREGATE in SQL. PostgreSQL comes with
> several aggregates that are nice to use when one needs to keep every entry
> in the output.

For a complete list of PostgreSQL Aggregates, the documentation is once
again all you need. Have a look at [Aggregate
Functions](https://www.postgresql.org/docs/current/static/functions-aggregate.html).

# PostgreSQL Arrays

PostgreSQL arrays can be built from the result of a query thanks to the
`array_agg()` aggregate. With that we can accumulate in a PostgreSQL arrays
all the incident status per driver in 2016:

~~~
  select format('%s %s', drivers.forename, drivers.surname) as name,
         sum(points) as points,
         
         array_agg(distinct status order by status)
            filter(
                where statusid is not null
                  and status <> 'Finished'
            ) as incidents
    
    from drivers
         join results using(driverid)
         join races using(raceid)
         join status using(statusid)
   
   where races.year = 2016

group by drivers.driverid
order by points desc;
~~~

The query might look complex. That's because we want for a real use case
with some complexity in it. We collect the result status in an array, and
filter out the *Finished* and NULL entries:

~~~
       name        │ points │                                   incidents                                    
═══════════════════╪════════╪════════════════════════════════════════════════════════════════════════════════
 Nico Rosberg      │    385 │ {Collision}
 Lewis Hamilton    │    380 │ {Collision,Engine}
 Daniel Ricciardo  │    256 │ {"+1 Lap"}
 Sebastian Vettel  │    212 │ {Collision,Engine,Tyre}
 Max Verstappen    │    204 │ {Accident,Gearbox,"Power Unit"}
 Kimi Räikkönen    │    186 │ {Accident,Collision,Engine,Wheel}
 Sergio Pérez      │    101 │ {"+1 Lap",Brakes}
 Valtteri Bottas   │     85 │ {"+1 Lap",Overheating,Suspension}
 Nico Hülkenberg   │     72 │ {"+1 Lap",Brakes,Collision,Engine}
 Fernando Alonso   │     54 │ {"+1 Lap",Battery,Collision,Engine,Gearbox}
 Felipe Massa      │     53 │ {"+1 Lap","+2 Laps",Brakes,Collision,Retired,Suspension}
 Carlos Sainz      │     46 │ {"+1 Lap","Collision damage",Puncture,Retired,Suspension}
 Romain Grosjean   │     29 │ {"+1 Lap","+2 Laps",Brakes,Engine,Retired,Withdrew}
 Daniil Kvyat      │     25 │ {"+1 Lap",Accident,Battery,Gearbox,Retired,Suspension}
 Jenson Button     │     21 │ {"+1 Lap",Brakes,"Collision damage","Oil leak","Power Unit",Suspension}
 Kevin Magnussen   │      7 │ {"+1 Lap","+2 Laps",Accident,Gearbox,"Power loss",Retired,Suspension}
 Felipe Nasr       │      2 │ {"+1 Lap","+2 Laps",Brakes,"Collision damage","Power Unit",Retired}
 Pascal Wehrlein   │      1 │ {"+1 Lap","+2 Laps",Brakes,Collision,"Collision damage","Oil leak","Spun off"}
 Jolyon Palmer     │      1 │ {"+1 Lap","+2 Laps",Accident,"Collision damage",Gearbox,Hydraulics,Retired}
 Stoffel Vandoorne │      1 │ {"+1 Lap"}
 Marcus Ericsson   │      0 │ {"+1 Lap","+2 Laps",Collision,Engine,Gearbox,"Power Unit"}
 Rio Haryanto      │      0 │ {"+1 Lap","+2 Laps","+4 Laps",Collision,Mechanical,"Spun off"}
 Esteban Gutiérrez │      0 │ {"+1 Lap","+2 Laps",Brakes,Collision,Retired,"Wheel nut"}
 Esteban Ocon      │      0 │ {"+1 Lap","+2 Laps"}
(24 rows)
~~~

We can see somewhat of a correlation in between the total number of points
and total number of incidents. It's like that was related, somehow.

# String Aggregates

With PostgreSQL we can also build a string rather than an array, if that
makes more sense for your application to process the values. Maybe it's only
printing them out:

~~~
  select format('%s %s', drivers.forename, drivers.surname) as name,
         sum(points) as points,
         
         string_agg(distinct status, ', ' order by status)
            filter(
                where statusid is not null
                  and status <> 'Finished'
            ) as incidents
    
    from drivers
         join results using(driverid)
         join races using(raceid)
         join status using(statusid)
   
   where races.year = 2016

group by drivers.driverid
order by points desc;
~~~

No surprises, this time we get the following strings:

~~~
       name        │ points │                                 incidents                                 
═══════════════════╪════════╪═══════════════════════════════════════════════════════════════════════════
 Nico Rosberg      │    385 │ Collision
 Lewis Hamilton    │    380 │ Collision, Engine
 Daniel Ricciardo  │    256 │ +1 Lap
 Sebastian Vettel  │    212 │ Collision, Engine, Tyre
 Max Verstappen    │    204 │ Accident, Gearbox, Power Unit
 Kimi Räikkönen    │    186 │ Accident, Collision, Engine, Wheel
 Sergio Pérez      │    101 │ +1 Lap, Brakes
 Valtteri Bottas   │     85 │ +1 Lap, Overheating, Suspension
 Nico Hülkenberg   │     72 │ +1 Lap, Brakes, Collision, Engine
 Fernando Alonso   │     54 │ +1 Lap, Battery, Collision, Engine, Gearbox
 Felipe Massa      │     53 │ +1 Lap, +2 Laps, Brakes, Collision, Retired, Suspension
 Carlos Sainz      │     46 │ +1 Lap, Collision damage, Puncture, Retired, Suspension
 Romain Grosjean   │     29 │ +1 Lap, +2 Laps, Brakes, Engine, Retired, Withdrew
 Daniil Kvyat      │     25 │ +1 Lap, Accident, Battery, Gearbox, Retired, Suspension
 Jenson Button     │     21 │ +1 Lap, Brakes, Collision damage, Oil leak, Power Unit, Suspension
 Kevin Magnussen   │      7 │ +1 Lap, +2 Laps, Accident, Gearbox, Power loss, Retired, Suspension
 Felipe Nasr       │      2 │ +1 Lap, +2 Laps, Brakes, Collision damage, Power Unit, Retired
 Pascal Wehrlein   │      1 │ +1 Lap, +2 Laps, Brakes, Collision, Collision damage, Oil leak, Spun off
 Jolyon Palmer     │      1 │ +1 Lap, +2 Laps, Accident, Collision damage, Gearbox, Hydraulics, Retired
 Stoffel Vandoorne │      1 │ +1 Lap
 Marcus Ericsson   │      0 │ +1 Lap, +2 Laps, Collision, Engine, Gearbox, Power Unit
 Rio Haryanto      │      0 │ +1 Lap, +2 Laps, +4 Laps, Collision, Mechanical, Spun off
 Esteban Gutiérrez │      0 │ +1 Lap, +2 Laps, Brakes, Collision, Retired, Wheel nut
 Esteban Ocon      │      0 │ +1 Lap, +2 Laps
(24 rows)
~~~

# JSON Aggregates

If you PostgreSQL driver in your programming language of choice makes it
harder than necessary for you to process PostgreSQL arrays, then don't
switch to using `string_agg()` already. Have a look at building a JSON array
instead, I'm sure you have all the needed facilities for processing that:

~~~
  select format('%s %s', drivers.forename, drivers.surname) as name,
         sum(points) as points,
         
         json_agg(distinct status order by status)
            filter(
                where statusid is not null
                  and status <> 'Finished'
            ) as incidents
    
    from drivers
         join results using(driverid)
         join races using(raceid)
         join status using(statusid)
   
   where races.year = 2016

group by drivers.driverid
order by points desc;
~~~

And there we have JSON output. Simple as that:

~~~
       name        │ points │                                         incidents                                         
═══════════════════╪════════╪═══════════════════════════════════════════════════════════════════════════════════════════
 Nico Rosberg      │    385 │ ["Collision"]
 Lewis Hamilton    │    380 │ ["Collision", "Engine"]
 Daniel Ricciardo  │    256 │ ["+1 Lap"]
 Sebastian Vettel  │    212 │ ["Collision", "Engine", "Tyre"]
 Max Verstappen    │    204 │ ["Accident", "Gearbox", "Power Unit"]
 Kimi Räikkönen    │    186 │ ["Accident", "Collision", "Engine", "Wheel"]
 Sergio Pérez      │    101 │ ["+1 Lap", "Brakes"]
 Valtteri Bottas   │     85 │ ["+1 Lap", "Overheating", "Suspension"]
 Nico Hülkenberg   │     72 │ ["+1 Lap", "Brakes", "Collision", "Engine"]
 Fernando Alonso   │     54 │ ["+1 Lap", "Battery", "Collision", "Engine", "Gearbox"]
 Felipe Massa      │     53 │ ["+1 Lap", "+2 Laps", "Brakes", "Collision", "Retired", "Suspension"]
 Carlos Sainz      │     46 │ ["+1 Lap", "Collision damage", "Puncture", "Retired", "Suspension"]
 Romain Grosjean   │     29 │ ["+1 Lap", "+2 Laps", "Brakes", "Engine", "Retired", "Withdrew"]
 Daniil Kvyat      │     25 │ ["+1 Lap", "Accident", "Battery", "Gearbox", "Retired", "Suspension"]
 Jenson Button     │     21 │ ["+1 Lap", "Brakes", "Collision damage", "Oil leak", "Power Unit", "Suspension"]
 Kevin Magnussen   │      7 │ ["+1 Lap", "+2 Laps", "Accident", "Gearbox", "Power loss", "Retired", "Suspension"]
 Felipe Nasr       │      2 │ ["+1 Lap", "+2 Laps", "Brakes", "Collision damage", "Power Unit", "Retired"]
 Pascal Wehrlein   │      1 │ ["+1 Lap", "+2 Laps", "Brakes", "Collision", "Collision damage", "Oil leak", "Spun off"]
 Jolyon Palmer     │      1 │ ["+1 Lap", "+2 Laps", "Accident", "Collision damage", "Gearbox", "Hydraulics", "Retired"]
 Stoffel Vandoorne │      1 │ ["+1 Lap"]
 Marcus Ericsson   │      0 │ ["+1 Lap", "+2 Laps", "Collision", "Engine", "Gearbox", "Power Unit"]
 Rio Haryanto      │      0 │ ["+1 Lap", "+2 Laps", "+4 Laps", "Collision", "Mechanical", "Spun off"]
 Esteban Gutiérrez │      0 │ ["+1 Lap", "+2 Laps", "Brakes", "Collision", "Retired", "Wheel nut"]
 Esteban Ocon      │      0 │ ["+1 Lap", "+2 Laps"]
(24 rows)
~~~

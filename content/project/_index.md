---
title: "Project Methodology"
description: "“A goal without a plan is just a wish” — Antoine de Saint-Exupéry"
featured_image: '/img/Train_wreck_at_Montparnasse_1895.jpg'

---

A migration project may take a little more than a week-end, so you need to
consider it as a real project. Follow this methodology and it's all going to
be fine, I promise:

{{< figure src="/img/keep-calm-and-automate-all-the-things-13.png"
         title="Keep Calm and Automate All the Things!" >}}

When migrating from MySQL to PostgreSQL, follow those steps:

  1. Setup your target PostgreSQL Architecture
  2. Fork a Continuous Integration environment that uses PostgreSQL
  3. Migrate the data over and over again every night, from production
  4. As soon as the CI is all green using PostgreSQL, schedule the D-Day
  5. Migrate without suprise and enjoy!
  
# PostgreSQL Architecture

That needs to be done first. Setup your *High Availability* before doing
anything else. The goal is for both the service and the data to be highly
available, so first setup a fully automated *backup and restore* solution.

For that, use either [pgbarman](http://www.pgbarman.org) or
[pgbackrest](http://pgbackrest.org), or maybe
[WAL-e](https://github.com/wal-e/wal-e) if you want to use Amazon S3.
Consider its replacement [WAL-G](https://github.com/wal-g/wal-g) too.

Don't roll it yourself. It's really easy to do it wrong, and what you want
is an all automated **recovery** solution. Go for that.

# Primary server and replicas

In PostgreSQL we chose to stop talking about slavery as being something
crucial to our daily lifes. So we setup High Availability with a *Primary*
server and a set of *Secondary* servers, also named *Standby* or *Replica*.

If you want to understand all the details about the setup, you can read the
whole PostgreSQL documentation about [High Availability, Load Balancing, and
Replication](https://www.postgresql.org/docs/current/static/high-availability.html)
and then about [Logical
Replication](https://www.postgresql.org/docs/current/static/logical-replication.html).

The PostgreSQL documentation best in class, and patches that add or modify
PostgreSQL features are only accepted when they also update all the impacted
documentation. Which means that our documentation is always uptodate and
trustworthy. That's why you don't see much comments in there. If any.

If you're not sure about what to do now, setup a PostgreSQL Hot Standby
physical replica by following the steps at [Hot
Standby](https://wiki.postgresql.org/wiki/Hot_Standby). It looks more
complex than it is. All you need to do is :

  1. check your `postgresql.conf` to allow for replication
  2. open replication privileges on the network in `pg_hba.conf`
  3. use `pg_basebackup` to have a remote copy of your *Primary* data
  4. start your *Replica* with a setup that connects to the *Primary*

That's it really.

# MySQL Row-Based Replication

In MySQL it's possible to setup replication and still edit local data in
other tables in the Replica. This is a solid use-case for some
architectures. Of course, as soon as the replica has a different data set
than the Primary, then the solution doesn't provide High Availability. You
might still be interested into having a similar solution in PostgreSQL.

Starting with PostgreSQL 10 you can use *Logical Replication*. It's easy to
setup, as seen in the [Logical Replication Quick
Setup](https://www.postgresql.org/docs/10/static/logical-replication-quick-setup.html)
part of the PostgreSQL documentation.

# CI/CD

That's how we do it now right? It's all automated everywhere into a Jenkins
or a Travis setup, or something equivalent. Or even something better. Well
then, do the same thing for your migration project.

# Nightly Data Migration

Chances are that once your data migration script is tweaked for all the data
you've seen through, some new data is going to show up in production that
will defeat your script.

To avoid data related surprises on the D-Day, just run the whole data
migration script for the production's data every night, for the whole
duration of the project. You will have such a track record of catering with
new data that you will fear no surprises. In a migration project, surprises
seldom are the good ones.

> _“If it wasn't for bad luck, I wouldn't have no luck at all.”_
>
> Albert King, Born Under a Bad Sign

# Porting the Code from MySQL to PostgreSQL

Now that you have a CI/CD environment fresh with yesterday's production data
every morning, it's time to rewrite those MySQL queries for PostgreSQL.
There are a couple things to know when doing that:

  - PostgreSQL likes JOINs
  
    Yes that's right, you can actually have very fast running queries using
    JOINs in PostgreSQL.
    
  - Quoting is different
  
    In PostgreSQL we quote SQL identifiers using "double quotes" and literal
    values using 'single quotes'.
    
  - `on duplicate key update` is written `on conflict do update`
  
    And it might even be `do nothing`, too.

  - See the [FAQ](/post) section for more conversion hints.

When you see differences in the SQL in between PostgreSQL and SQL, mostly it
is PostgreSQL following the SQL standard syntax.

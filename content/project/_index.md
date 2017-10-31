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

# CI/CD

That's how we do it now right? It's all automated everywhere into a Jenkins
or a Travis setup, or something equivalent. Or even something better. Well
then, do the same thing for your migration project.

# Nightly data migration from production

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


---
title: "About"
description: "How to Migrate from MySQL to PostgreSQL?"
featured_image: "/img/thompson-history-of-composing-machines-1200grey-0078-wicks-rotary-typecaster-crop-4026x4224-scale-1024x1074.jpg"

---

When migrating from MySQL to PostgreSQL, you need to implement a proper
[Project Methodology](/project). Your migration budget is then splitted in
those 4 areas:

  1. Migrating the Data
  2. Migrating the Code
  3. Migrating the Service
  4. The Opportunity Cost
  
# Migrating the Data

[pgloader](http://pgloader.io) implements fully automated migration from
MySQL to PostgreSQL in a single command line:

~~~ bash
$ pgloader mysql://user@host/dbname pgsql://user@host/dbname
~~~

{{< figure src="/img/TypeCaster.jpg"
         title="A Type Caster" >}}


More complex scenarios are of course possible:

~~~ bash
$ pgloader f1db.load
~~~

With the following `f1db.load` command:

~~~ bash
load database
  from mysql://root@localhost/f1db
  into pgsql:///f1db
  with concurrency = 2,
       multiple readers per thread,
       rows per range = 50000
       prefetch rows = 1000000;
~~~

The result then looks like the following on my laptop:

~~~
LOG Data errors in '/private/tmp/pgloader/'
LOG Parsing commands from file #P"/Users/dim/dev/pgloader/test/mysql/f1db.load"
LOG report summary reset
               table name     errors       rows      bytes      total time
-------------------------  ---------  ---------  ---------  --------------
          fetch meta data          0         33                     0.461s
           Create Schemas          0          0                     0.001s
         Create SQL Types          0          0                     0.007s
            Create tables          0         26                     0.218s
           Set Table OIDs          0         13                     0.007s
-------------------------  ---------  ---------  ---------  --------------
            f1db.circuits          0         73     8.5 kB          0.035s
  f1db.constructorresults          0      11052   184.6 kB          0.212s
        f1db.constructors          0        208    15.0 kB          0.026s
             f1db.drivers          0        840    79.6 kB          0.052s
            f1db.laptimes          0     417743    10.9 MB          5.025s
f1db.constructorstandings          0      11806   247.3 kB          0.226s
     f1db.driverstandings          0      31509   714.0 kB          0.886s
            f1db.pitstops          0       5927   198.7 kB          0.410s
               f1db.races          0        976    98.4 kB          0.304s
             f1db.seasons          0         68     3.9 kB          0.145s
          f1db.qualifying          0       7337   279.0 kB          0.216s
             f1db.results          0      23597     1.3 MB          0.688s
              f1db.status          0        134     1.7 kB          0.040s
-------------------------  ---------  ---------  ---------  --------------
  COPY Threads Completion          0          4                     5.626s
           Create Indexes          0         20                     2.548s
   Index Build Completion          0         20                     1.150s
          Reset Sequences          0         10                     0.078s
             Primary Keys          0         13                     0.020s
      Create Foreign Keys          0          0                     0.000s
          Create Triggers          0          0                     0.000s
         Install Comments          0          0                     0.000s
-------------------------  ---------  ---------  ---------  --------------
        Total import time          ✓     511270    14.0 MB          9.422s
~~~

As the goal for [pgloader](http://pgloader.io) is for the migration to be
fully automated and part of your CI/CD nightly jobs, it is also possible to
add type casting rules to the command.

Here's a more complex example:

~~~ sql
load database
     from mysql://root@unix:/tmp/mysql.sock:3306/pgloader
     into postgresql://dim@localhost/pgloader

 alter schema 'pgloader' rename to 'mysql'

 CAST column base64.id to uuid drop typemod drop not null,
      column base64.data to jsonb using base64-decode,

      type decimal when (and (= 18 precision) (= 6 scale))
        to "double precision" drop typemod

 before load do $$ create schema if not exists mysql; $$;
~~~

In this example you can see that we chose to cast the column `id` of the
`base64` table to the `uuid` type in PostgreSQL. Read the [pgloader
reference documentation](http://pgloader.io/howto/pgloader.1.html) for all
the details about the casting rules.

# Migrating the Code

First setup a CI/CD fork using PostgreSQL, then see the [FAQ](/post/) here
for more details about that step.

# Migrating the Service

This needs to be carefully planned to. Check about your allowed downtime and
if the migration runs fully within that window, as it's the easier to
implement, always.

Other option such as incremental migration or replication from MySQL to
PostgreSQL are possible. They come with complexity that is best avoided,
though. Keep it as Simple as you can.

Often enough, the source database contains tables that receive very low DML
trafic (that's `insert`, `update`, and `delete`). Then you can migrate those
tables ahead of schedule, and only migrate the *live* tables during the
downtime window.

To implement that, [pgloader](http://pgloader.io) implements the following
clauses in its command language:

~~~
INCLUDING ONLY TABLE NAMES MATCHING ~/film/, 'actor'
EXCLUDING TABLE NAMES MATCHING ~<ory>
~~~

Use that to split your migration into the *archive* parts and the *live*
parts if you have to.

# Opportunity Cost

While you're working on the migration, you are not implementing those new
shiny features that the Product department is waiting for. Everybody needs
to understand that you're busy migrating, and that the new system is going
to be so much better off using PostgreSQL.


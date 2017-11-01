---
featured_image: "/img/mackellar-american-printer-1882-pivotal-caster-1200rgb-2048x.jpg"
title: "How can I prevent SQL injection in PHP?"
description: "Little Bobby Tables is fun and all that, but still…"
---

> When using PHP, be sure to use the
> [pg_query_params](http://php.net/manual/function.pg-query-params.php)
> function, or an higher-level API that then uses that PostgreSQL facility.
> Then you're protected against any and all SQL injection attacks, by
> design.

An *SQL Injections* is a security breach, one made famous by the [Exploits
of a Mom](https://xkcd.com/327/) `xkcd` comic episode in which we read about
*little Bobby Tables*.

{{< figure src="/img/exploits_of_a_mom.png"
          link="https://xkcd.com/327/"
         title="Exploits of a Mom" >}}

PostgreSQL implements a protocol level facility to send the static SQL query
text separately from its dynamic arguments. An SQL injection happens when
the database server is mistakenly led to consider a dynamic argument of a
query as part of the query text. Sending those parts as separate entities
over the protocol means that SQL injection is no longer possible.

The PostgreSQL protocol is fully documented and you can read more about
*extended query* support on the [Message
Flow](https://www.postgresql.org/docs/current/static/protocol-flow.html)
documentation page. Also relevant is the `PQexecParams` driver API,
documented as part of the [command execution
functions](https://www.postgresql.org/docs/current/static/libpq-exec.html)
of the `libpq` PostgreSQL C driver.

A lot of PostgreSQL application drivers are based on the libpq C driver,
which implements the PostgreSQL protocol and is maintained alongside the
main server's code. Some drivers variants also exist that don't link to any
C runtime, in which case the PostgreSQL protocol has been implemented in
another programming language. That's the case for variants of the JDBC
driver, and the `pq` Go driver too, among others.

It is advisable that you read the documentation of your current driver and
understand how to send SQL query parameters separately from the main SQL
query text; this is a reliable way to never have to worry about *SQL
injection* problems ever again.

In particular, ***never*** build a query string by concatenating your query
arguments directly into your query strings, i.e. in the application client
code. Never use any library, ORM or another tooling that would do that. When
building SQL query strings that way, you open your application code to
serious security risk for no reason.

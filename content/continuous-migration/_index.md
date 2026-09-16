---
title: "What Is Continuous Migration, and Why Does It Work?"
eyebrow: "A Concept, Not a Checklist"
icon: "fa-solid fa-arrows-rotate"
description: "The idea behind treating a database migration as an ongoing pipeline instead of a single cutover event — and why it changes how much risk the project actually carries."
---

A migration to PostgreSQL is a real project, and some risk needs to be
handled properly. The method most of this site describes for handling
that risk is called **Continuous Migration**. This page isn't the
step-by-step of how to run it — that's the [Methodology](/project/) page.
This is the shorter question underneath it: what is it, actually, and why
does it hold up better than the alternative?

# What it is

Continuous Migration is comparable to continuous integration and
continuous deployment — CI/CD, applied to the migration itself rather
than to application code. The core idea: set up a target PostgreSQL
environment first, then use it every day as developers work on porting
the software to it. As soon as a PostgreSQL environment exists, it
becomes possible to fork a CI/CD setup onto the PostgreSQL branch of the
code repository. In parallel, the data keeps getting migrated over and
over — not once, but continuously — so the target environment is never
stale by more than a day.

# Why it works

The alternative is the big-bang migration: freeze, migrate everything in
one pass, cut over, hope. Continuous Migration exists because that
alternative concentrates all of a project's risk into a single event, at
the exact moment the least is known about what will actually go wrong.

Splitting a migration project into chunks and allowing incremental
progress does three things a single cutover can't:

- **It surfaces problems early, while they're still cheap.** A schema
  edge case, an encoding mismatch, a query that behaves differently under
  PostgreSQL's planner — these get found on an ordinary Tuesday against a
  CI pipeline, not during a maintenance window with a rollback clock
  running.
- **It builds a track record, not a guess.** By the time D-Day is
  scheduled, the migration has already run successfully many times over
  real, current production data. The decision to cut over is backed by
  evidence, not confidence.
- **It lets the project pause and resume.** Migrations get
  deprioritized, resourced, and reprioritized like any other project.
  A continuous process degrades gracefully when that happens; a
  half-finished big-bang migration usually doesn't.

# What it isn't

It isn't a tool, and it isn't a specific pipeline configuration — those
are the *how*, covered on the [Methodology](/project/) page, with
pgloader and pg_auto_failover as the concrete pieces that make it
practical. This page is just the idea underneath all of that: migrate
continuously, not once, so the risk gets spent gradually instead of all
at once on a single night.

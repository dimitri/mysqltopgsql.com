---
title: "The Continuous Migration Method"
eyebrow: "5 Steps, No Surprises on D-Day"
icon: "fa-solid fa-route"
description: "“A goal without a plan is just a wish” — set up the target architecture, fork CI onto it, migrate nightly, then cut over once CI stays green."
---

<section class="section">
  <div class="wrapper">
    <div class="content">
      <h2 class="h-lead"><i class="fa-solid fa-list-ol section-icon"></i>The 5-step method</h2>
      <p class="subtitle">Split a migration project into chunks, not one big-bang cutover</p>
      <p>A migration project may take a little more than a weekend, so treat
      it like a real project. This method — <a href="/continuous-migration/">Continuous
      Migration</a> — is how to run it without betting everything on a
      single cutover night. Follow it and it's all going to be fine, I
      promise:</p>
      <!-- This is a real sequence: each step's output is the next step's
           input, and doing them out of order is the failure mode the whole
           method exists to prevent. Five equal cards said the opposite —
           five parallel options, pick some. A linear numbered strip encodes
           the order in the layout itself. (HTML comment, not a Go template
           comment: content files aren't run through the template engine,
           so {{/* */}} would render literally here.) -->
      <ol class="steps-linear">
        <li>
          <span class="step-n">01</span>
          <h3>Target architecture</h3>
          <p>Stand up your target PostgreSQL architecture — HA included — before migrating a single row.</p>
        </li>
        <li>
          <span class="step-n">02</span>
          <h3>Fork CI onto it</h3>
          <p>Fork a Continuous Integration environment that runs your full test suite against PostgreSQL.</p>
        </li>
        <li>
          <span class="step-n">03</span>
          <h3>Migrate nightly</h3>
          <p>Re-run the full data migration from production every night, for the length of the project.</p>
        </li>
        <li>
          <span class="step-n">04</span>
          <h3>Go green</h3>
          <p>Schedule D-Day only once CI has run clean on PostgreSQL for long enough to trust it.</p>
        </li>
        <li>
          <span class="step-n">05</span>
          <h3>Cut over</h3>
          <p>Migrate on D-Day — by now it&rsquo;s a formality, not a leap of faith.</p>
        </li>
      </ol>
    </div>
  </div>
</section>

<section class="section section-alt">
  <div class="wrapper">
    <div class="content">
      <h2><i class="fa-solid fa-server section-icon"></i>PostgreSQL architecture</h2>
      <p class="subtitle">High availability comes first, not last</p>
      <p>That needs to be done first. Setup your <em>High Availability</em>
      before doing anything else. The goal is for both the service and the
      data to be highly available, so start with a fully automated
      <em>backup and restore</em> solution.</p>
      <p>For backup/restore, use either <a href="http://www.pgbarman.org">pgbarman</a>
      or <a href="http://pgbackrest.org">pgbackrest</a>, or
      <a href="https://github.com/wal-g/wal-g">WAL-G</a> if you want
      S3-compatible object storage.</p>
      <p>For automated failover, use
      <a href="https://github.com/hapostgres/pg_auto_failover">pg_auto_failover</a> —
      a monitor-node-based failover extension and service: one monitor, N
      Postgres nodes, automated promotion on failure, no external consensus
      cluster to run and babysit separately.</p>
      <p>Don't roll either of these yourself. It's really easy to do it
      wrong, and what you want is an all-automated <strong>recovery</strong>
      solution. Go for that.</p>
    </div>
  </div>
</section>

<section class="section">
  <div class="wrapper">
    <div class="content">
      <h2><i class="fa-solid fa-arrows-rotate section-icon"></i>Primary server and replicas</h2>
      <p class="subtitle">The shape every later module builds on</p>
      <p>Set up High Availability with a <em>Primary</em> server and a set
      of <em>Secondary</em> servers, also named <em>Standby</em> or
      <em>Replica</em>. Read the whole PostgreSQL documentation about
      <a href="https://www.postgresql.org/docs/current/static/high-availability.html">High
      Availability, Load Balancing, and Replication</a>, and then about
      <a href="https://www.postgresql.org/docs/current/static/logical-replication.html">Logical
      Replication</a>.</p>
      <div class="callout">
        <i class="fa-solid fa-triangle-exclamation callout-icon"></i>
        <div>
          <strong>Coming from MySQL:</strong> MySQL's row-based replication
          lets a replica keep accepting local writes to other tables even
          while replicating — a real use case for some architectures, but
          the moment a replica's data diverges from the primary, it's no
          longer providing High Availability, just a copy that drifted.
          PostgreSQL's model doesn't allow that ambiguity: a physical
          Secondary is read-only, full stop — every write goes through the
          Primary, so there's never a question of which copy is correct.
          If your MySQL setup relied on writable replicas, that's
          architecture to redesign here, not just syntax to port.
        </div>
      </div>
      <figure>
        <img src="/images/pg_auto_failover-arch.png" alt="pg_auto_failover architecture: Application, Primary, Secondary, and Monitor">
        <figcaption>pg_auto_failover's architecture: a monitor node watches Primary and Secondary over Postgres streaming replication, and drives promotion on failure.</figcaption>
      </figure>
      <p>If you're not sure what to do now, don't hand-roll the replication
      config — use <a href="https://github.com/hapostgres/pg_auto_failover">pg_auto_failover</a>
      instead. Two commands get you there:</p>
      <ol>
        <li>Create the monitor node: <code>pg_autoctl create monitor</code>.</li>
        <li>Register your Primary and Secondary against it: <code>pg_autoctl create postgres</code>, run on each node.</li>
      </ol>
      <p>That's it — the monitor handles the replication setup, health
      checks, and promotion on failure for you, instead of you assembling
      <code>postgresql.conf</code>, <code>pg_hba.conf</code>, and
      <code>pg_basebackup</code> by hand.</p>
    </div>
  </div>
</section>

<section class="section section-alt">
  <div class="wrapper">
    <div class="content">
      <h2><i class="fa-solid fa-code-branch section-icon"></i>CI/CD for your migration</h2>
      <p class="subtitle">Treat the migration itself as a pipeline, not a one-shot script</p>
      <p>You already run CI/CD for application code — validate, promote,
      approve, record. Apply the same discipline to the migration itself:
      the whole point of nightly re-migration (next section) is to put the
      migration on the same automated, continuously-validated footing as
      any other change, so nothing about D-Day is a surprise.</p>
    </div>
  </div>
</section>

<section class="section">
  <div class="wrapper">
    <div class="content">
      <h2><i class="fa-solid fa-moon section-icon"></i>Nightly data migration</h2>
      <p class="subtitle">Run the whole thing, every night, for the length of the project</p>
      <p>Chances are that once your data migration script is tweaked for
      all the data you've seen, some new data will show up in production
      that defeats your script.</p>
      <p>To avoid data-related surprises on D-Day, run the whole data
      migration script against production data every night, for the whole
      duration of the project. You'll build such a track record handling
      new data that you'll fear no surprises. In a migration project,
      surprises are seldom the good kind.</p>
      <blockquote>
        <p><em>&ldquo;If it wasn't for bad luck, I wouldn't have no luck at all.&rdquo;</em></p>
        <p class="cite">Albert King, Born Under a Bad Sign</p>
      </blockquote>
    </div>
  </div>
</section>

<section class="section section-alt">
  <div class="wrapper">
    <div class="content">
      <h2><i class="fa-solid fa-code section-icon"></i>Porting the code from MySQL to PostgreSQL</h2>
      <p class="subtitle">What actually changes in your queries</p>
      <p>Now that you have a CI/CD environment fresh with yesterday's
      production data every morning, it's time to rewrite those MySQL
      queries for PostgreSQL. A few things to know:</p>
      <ul>
        <li><strong>PostgreSQL likes JOINs</strong> — yes, really, you can have very fast queries using JOINs in PostgreSQL.</li>
        <li><strong>Quoting is different</strong> — PostgreSQL quotes SQL identifiers using <code>"double quotes"</code> and literal values using <code>'single quotes'</code>.</li>
        <li><code>on duplicate key update</code> is written <code>on conflict do update</code> — and it might even be <code>do nothing</code>.</li>
        <li>See the <a href="/post/">FAQ</a> for more conversion hints.</li>
      </ul>
      <p>Most differences you'll see are PostgreSQL simply following the
      SQL standard more closely.</p>
    </div>
  </div>
</section>

---
layout: post
title: "PostgreSQL in 2026: The Default Choice for Modern Data Infrastructure"
description: "Why PostgreSQL is the default database in 2026: JSONB, extensions, cloud and pgvector, plus how teams manage schema changes, performance and migrations."
tags:
  - postgresql
  - postgres
  - database
---

[PostgreSQL performance](https://www.devart.com/dbforge/postgresql/studio/postgresql-performance-tuning-and-optimization.html) is easy to benchmark, but speed alone does not explain its popularity. What makes this database attractive comes down to several practical considerations.

PostgreSQL handles relational data well, supports SQL properly, and provides JSONB and a long list of extensions when the original setup is no longer enough. It’s also available across the major cloud platforms. A team can start with a normal application database and keep building on it before it needs to introduce another system.

That’s why today, about 58.2% of professional developers use this tool according to the [2025 Stack Overflow Developer Survey](https://survey.stackoverflow.co/2025/technology).

However, while PostgreSQL can play a broader role than most teams initially expect, every added workload creates something else to monitor, tune and maintain. Application queries, reports, data pipelines, schema changes, extensions and AI-related features may all end up relying on the same database.

The important question in 2026 is no longer only what PostgreSQL can handle. It is whether teams can manage the complexity that comes with putting so much in one place.

## Why has PostgreSQL become the developer default?

PostgreSQL is a typical place to start. It does what most applications need most of the time: storing related records, enforcing rules, and querying them with SQL. You don’t need the team to know what the product will look like in two years to select it.

Imagine a product catalog. Prices and stock levels should be in neat columns. Attributes can be less co-operative. A shirt has a size, a laptop has memory and tomorrow’s product may have fields we never thought of. JSONB gives developers a place to store those differences without having to set up a document database immediately.

Of course, there are other reasons to go with PostgreSQL. Developers are aware of this, hosting is readily available and its ecosystem provides capabilities that would otherwise have to be provided by different services.

The [2025 database PostgreSQL results from Stack Overflow](https://survey.stackoverflow.co/2025/technology) show just how common that starting point has become. If the application is specialized in nature, the starting point for the decision may be different.

### Open-source maturity makes PostgreSQL more predictable

With nearly 30 years of open-source development, PostgreSQL has a pretty sizeable track record to draw from. Engineers can plan maintenance and investigate problems without relying solely on one vendor, with published support policies and a documented release process.

Maturity is not the end of development. PostgreSQL 18 added asynchronous I/O, skip scans for multicolumn B-tree indexes and faster-running upgrade checks. The project saw 3x gains on some storage-read tests with the new I/O sub-system. These are not predictions for each and every application.

The speed number gets attention but the upgrade enhancements are also important. Engineers working on a production database have to work around the application. They get less coverage than a benchmark, but faster checks and upgrades to the process help that maintenance work.

Upgrades are not a sure thing, nor do incidents necessarily result in damage. This provides the change planner a better basis for deciding what to test and what could go wrong.

### One engine can hold more than one kind of truth

PostgreSQL supports flexible data without forcing developers to give up relational tables and constraints. The JSONB is useful when the attributes vary from record to record; accounts, payments and permissions can maintain a more explicit structure.

It’s a distinction worth maintaining. A field should be jsonb when someone understands what it means. When the application code expects a certain value to be there and valid, the team has to decide where to enforce that rule.

Arrays, ranges, generated columns, and full-text search provide more options. Extensions have spatial and vector capabilities. A small search capability could easily fit into the existing database without needing to provision or sync a separate service.

The savings are very real – in operational terms. That’s one less copy of the data to maintain, and one less set of credentials to manage. But extensions need to be checked out for upgrades and cloud migrations. They are part of the architecture, not an installation detail you can delegate to deployment.

## How does PostgreSQL fit cloud, AI, and analytics workflows?

Managed services offer several options for running PostgreSQL that can support reporting, pipelines, and AI-related features alongside application data. With a change in hosting much of the SQL and application code can be preserved.

A PostgreSQL cloud database could be a managed service from AWS, Azure or Google Cloud. Self-managed installs and Kubernetes operators are also available for private-cloud and hybrid deployments. Aurora PostgreSQL and AlloyDB take this a step further: they are still PostgreSQL compatible, but with a different storage and infrastructure underneath.

Nubank is a good example of how to change that operating model. NuPay migrated 7.5 TB and over 31 billion rows from self-managed PostgreSQL to Aurora PostgreSQL. AWS and Nubank claim to have cut their infrastructure costs by 25% following the move.

That’s from one migration, not a blanket cloud discount. A PostgreSQL cloud database therefore gives teams several operating options. But compatibility is not the same as identical behavior. Extensions, performance and maintenance processes can still vary between providers.

### Vector support arrived before the AI rush

Vector support reached the PostgreSQL ecosystem in 2021, before the recent rush to add generative AI to applications. The pgvector project's first release, version 0.1.0, arrived on April 20 that year.

The difference is important: pgvector provides the vector data type as an extension. This is not a native basic type in PostgreSQL. With the extension installed, developers are able to store embeddings on top of the records they describe and use similarity search to power recommendations, semantic search, retrieval-augmented generation, and other AI-powered features.

Bringing those records together can help implement early. The embedding of a document, its metadata and the application access rules do not have to be coordinated immediately between the two data bases. The retrieval query still needs to correctly implement those rules.

Some PostgreSQL-compatible deployments now handle much larger workloads. Google says AlloyDB customers are searching datasets from 100 million to [over a billion vectors](https://cloud.google.com/blog/products/databases/how-scann-for-alloydb-vector-search-compares-to-pgvector-hnsw). Those figures concern AlloyDB vector search, not pgvector generally. Don’t use them to size a standard PostgreSQL installation without testing.

### Analytics is moving closer to operational data

PostgreSQL analytics is good for reports and product metrics that need fresh operational data. There is no need for a warehouse behind a dashboard for today's orders.

The problem is when reporting interferes with the application. Queries that aggregate large tables may also need memory and I/O for customer requests. This can be right, helpful and a bad neighbor.

Some of the work can be offloaded to read replicas. Materialized views don't need to compute the same result on every request. "Refreshing" them has a cost. Other options for organizing or moving the data as reporting expands are partitioning, logical replication and downstream pipelines.

These options are expanding with cloud variants. Google said in tests, its AlloyDB columnar engine can process analytical queries up to 100 times faster than PostgreSQL alone. That figure applies to AlloyDB and the workloads Google tested.

For an existing application the immediate question is less: what happens to customer queries while the report is running? That metric is more useful than simply presuming the warehouse or the primary database has to be the answer.

## Why is managing PostgreSQL becoming the bigger challenge?

Managing PostgreSQL gets tough when multiple teams are using PostgreSQL and no one has the full picture of what’s changing. A healthy server won’t tell you whether an analyst’s report is based on a column that a developer is about to remove.

Developers, analysts, platform engineers, data engineers can all be doing reasonable work on their own. The problem lies where their duties overlap. Say a deployment passes in staging, but staging has no production view or a different index.

This makes release planning a part of PostgreSQL schema management. This includes dependencies, comparing environments, and figuring out who will review proposed changes. Indexes need similar care: adding one to fix a slow endpoint also adds to storage and write maintenance costs.

SQL console is not enough for more complex queries, reporting and data movement which means more to coordinate. At least someone should be able to say who owns a change, where was it tested and what actually made it to production. When something is broken, it’s harder to piece those answers back together.

### Performance problems usually come from the workload

A query's performance depends on the data and conditions in which it runs. Testing the same SQL with different parameters, stale statistics or concurrent writes can produce a very different result.

The [Nubank migration results](https://aws.amazon.com/blogs/database/migrating-mission-critical-payments-at-nubank-to-amazon-aurora-postgresql/) make this unusually clear. One historical receipts query dropped from more than 13 minutes to 0.424 seconds, roughly a 1,900-fold improvement. Average production improvements were 15% to 20%. Another broad query improved by a factor of 1.26.

The spectacular result is worth reporting, but it tells an engineer little about what their own workload might look like. That needs measurements.

`EXPLAIN` describes the approach the planner has taken. `EXPLAIN ANALYZE` needs care, too, when the statement modifies data or is expensive to run, as it adds evidence from running the query. Check representative parameters and concurrency and keep the before and after results. If a fix involves an index, then measure the impact on writes as well. Otherwise, a successful query optimization might leave the application with a different bottleneck.

### Database changes need more than one clever script

A PostgreSQL migration is easier to diagnose when data movement, replication, validation and cutover are treated as separate stages. If one stage fails, the team needs to know what has already completed before deciding what to do next.

Fluent Commerce's upgrade involved more than 350 Aurora PostgreSQL production databases, some as large as 32 TB. Its [published case study](https://aws.amazon.com/blogs/database/fluent-commerces-approach-to-near-zero-downtime-amazon-aurora-postgresql-upgrade-at-32-tb-scale-using-snapshots-and-aws-dms-ongoing-replication/) describes snapshots, restores, logical replication, AWS Database Migration Service and validation before cutover.

That scale is faced by few teams. But even a much smaller move raises awkward questions. How do you verify completeness? When does the application write to the new db? If rollback is necessary, what happens to the writes accepted after the cutover?

These are questions to answer in the plan before you start the migration. For schema only changes the work may be simpler: compare structures, look at the script and its dependencies and then rehearse against representative data. Check the resulting state after execution. A successful command is good evidence, but it is not the whole migration check.

## Where does dbForge Studio for PostgreSQL help?

dbForge Studio for PostgreSQL brings SQL editing, query profiling, schema comparison and data work into one environment. That can help a developer investigate a problem and prepare a change without repeatedly switching applications.

Take a slow query. Query Profiler displays the execution plan and profiling results. After editing the SQL, the developer can run it again and compare the results. When the problem sits inside a function, procedure or trigger, the [PostgreSQL debugger](https://www.devart.com/dbforge/postgresql/studio/postgresql-debugger.html) lets the developer set breakpoints, step through the code, watch variable values and pinpoint the statement that causes the problem. For schema work, comparisons between source and target databases can reveal environment differences and produce synchronization scripts for review.

Devart describes the product as a cross-platform [PostgreSQL GUI](https://www.devart.com/dbforge/postgresql/studio/) and IDE. Import, export and reporting are available too, although feature availability depends on the edition.

When evaluating a PostgreSQL IDE, I'd start with an ordinary task from the team's backlog. Can the next reviewer follow what changed? A PostgreSQL development tool earns its place there. The practical value of a PostgreSQL GUI tool is easier inspection of the database and generated SQL, especially when the work must be handed to someone else.

### AI-generated SQL still needs evidence

AI generated SQL must be validated as correct and performant, the same as SQL written by a developer. Half the battle is just running a statement.

Context-aware dbForge AI Assistant can generate queries, explain code, troubleshoot SQL and offer optimizations. That's good for first draft or unnamed statement. The reviewer still needs to verify the query returns the correct records and complies with the application’s permissions.

A join can look reasonable, and multiply rows unexpectedly. An aggregate can count the wrong thing and come up with a plausible number. Good formatting and confident explanations will not solve either problem.

Another way to build joins, columns and filters without losing the ability to inspect generated SQL, is to use a PostgreSQL visual query builder. Query Profiler then shows the evidence of execution for the tested run. These tools can be used to investigate the structure and cost of the query between them. We still have to look at the production load, locking behavior and representative data before we go live with the change.

### Automate work that has already been tested

PostgreSQL automation is good for repetitive tasks with known inputs and checks. It requires a schedule, and before that an unclear synchronization operation needs investigation.

[dbForge Studio for PostgreSQL](https://www.devart.com/dbforge/postgresql/studio/) provides command-line schema and data comparison, import and export, data generation and reporting. Re-use saved comparison settings in scheduled jobs or CI/CD workflows. This means that it is possible to do a comparison before a deployment and leave the output for review.

The review has to mean something. If the automated comparison says an object should be deleted, someone has to determine if the deletion is intentional. That the command passes the exit-code check does not answer that question.

Do not hardcode credentials in scripts. Log what ran, on what environment and if it completed. If wanted the generated SQL can be versioned with the change. Production operations that could potentially delete data or block application work may need approval steps.

Once these decisions are made, automation takes over the repetitive setup and run. If you find that the same job still needs manual correction, go back to the previous job before you add more environments to the schedule.

## Takeaway

PostgreSQL's appeal comes from a dependable relational database, a mature open-source project and enough flexibility to accommodate changing requirements. Developers can use SQL they know, add extensions where needed and choose among several hosting arrangements.

That combination explains why it so often enters a project before the architecture is fully settled. A team can build the application, learn from actual usage and make more specific infrastructure decisions later.

The word “later” requires some thought. One department added to a report can be a dependency for many others. A small search feature can become a large burden. By then you need to know who uses it and how fresh the data has to be to move it.

A sane PostgreSQL strategy includes those conversations while the system is still manageable. Review query plans, schema changes and operating costs for growth in workload. Make sure ownership is clear enough for someone to make the call to split a workload if necessary. PostgreSQL provides a team with tons of space to build. But you still need to manage that space.

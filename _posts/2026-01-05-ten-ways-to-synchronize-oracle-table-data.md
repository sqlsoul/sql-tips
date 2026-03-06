---
layout: post
title: "10 Ways to Synchronize Oracle Table Data"
description: "Ten ways to synchronize Oracle database tables, comparing manual methods, Oracle replication technologies, and third-party synchronization tools."
tags:
  - data compare
  - Oracle
---

In the process of developing databases many developers and testers encounter a problem of synchronizing data between local and remote [Oracle database](https://www.devart.com/dbforge/oracle/all-about-oracle-database/). Changes made in a local database must be reflected in a remote database. It's also necessary to check test results with the model database, to find non-synchronized tables that appeared after these tests, to have possibility to return the test database to its initial state when testing newly developed versions of databases intensively. Another problem may be to create CRUD operations quickly without making too much effort to find different objects and to write DML statements. Now let's find out how one can synchronize data of Oracle databases.

**Simple solutions:**

* Writing batch scripts manually
* Using database triggers

**Using solutions provided by Oracle:**

* Oracle Basic Replication
* Oracle Advanced Replication
* Oracle Streams
* Oracle GoldenGate
* Oracle DBMS\_COMPARISON package

**Using third-party replication solutions:**

* SharePlex for Oracle by Quest Software
* WisdomForce Database Sync System
* SymmetricDS by symmetricds.codehaus.org

## Writing Batch Scripts manually

The way of creating **update scripts** and executing them manually after every update performed to the database, is, of course, the simplest way and does not require any environment settings. But the process of detecting changes made to the database, creating the **CRUD operations** and the synchronization process itself stably takes a long time and in future it takes even longer when the created database grows.

## Using Database Triggers

This way of synchronization supposes that additional standard views will be created for each table at the remote site that would query the local database and also that an **INSTEAD OF trigger** will be created over these views to handle DML operations. Also you could create DATABASE LINK between your two databases. This would be the simplest way for synchronization (less code, less maintenance). There will be no problems until data that is already available on the side where changes are made is added on the side that receives the changes. In this case the trigger will show an error message and you will have to search for the conflicting records in these tables manually.

## Oracle Basic and Advanced Replication

Replication is the process of creating and maintaining replica versions of tables in a distributed database system. Oracle Server supports two different forms of replication.

**Basic replication** is implemented using standard CREATE SNAPSHOT or CREATE MATERIALIZED VIEW statements. It can only replicate data, replication is always one-way, and snapshot copies are read only. Oracle 8i implements snapshots and materialized views as a single table, previous releases implemented it as a view with an underlying SNAP$\_% table. Start by creating an optional snapshot log on the master database. If you do not want to do fast refreshes, you do not need to create a log. Also note that fast refreshes are not supported for complex queries.

**Advanced replication** supports various configurations of updatable-snapshot, multi-master, and update anywhere replication. It is more difficult to configure but allows data and other database objects like indexes and procedures to be replicated. Advanced Replication can be implemented on any application without having to make application changes. Advanced Replication is available only on Enterprise Server. To learn more about this way, refer to Oracle Database Advanced Replication manual.

Unfortunately, none of the replications support synchronization of the [LONG and LONG RAW data types](https://blog.devart.com/oracle-data-types.html), and also it's impossible to set Oracle replication for the tables that don't have Primary Key.

## Oracle Streams and GoldenGate

Unlike Advanced Replication, replication in **Oracle Streams** does not require creation of special structures in databases (journal tables, materialized views). Oracle Streams is based on processing information from database journal. As of Oracle version 9.2, Oracle Corporation makes Oracle Streams available on Oracle Enterprise Edition systems only. But in some releases of Oracle databases support for index-organized tables (IOTs) is not available as well as support for tables with function-based and descending indexes, LOB columns. Besides, there is no table comparison mechanism in Oracle Streams.

Oracle Corporation is now encouraging customers to use **GoldenGate** rather than Streams with new applications. GoldenGate's best-in-class solutions enable real-time data integration and continuous data availability by capturing and delivering updates of critical information as the changes occur and providing continuous data synchronization across heterogeneous environments. Besides, Oracle GoldenGate offers a high-speed program module **Oracle GoldenGate Veridata** that is designed to compare data quickly. The module finds differences in data in different databases and generates reports without influencing their accessibility. The shortcomings of this solution are the difficulty of configuration (it has it's own command, error code which is not SQLs or is not similar to it) and that errors are documented poorly. But all the same the price of this solution is quite high.

## Oracle DBMS\_COMPARISON package

The new feature of Oracle 11g is **DBMS\_COMPARISON package**. It helps to detect how data of 2 tables differ and synchronize the data if it is needed. When you compare an object with DBMS\_COMPARISON, you will compare an object in a local schema and a remote schema. The remote schema can be on the same database, or it can be on a distributed database, accessible through a database link. But this method also has its limitations – one of the servers should be of the 11g version and the other not lower than the 10.2 version, the server encodings should be identical, and not all data types can be compared and synchronized.

## Third-Party Replication Solutions

Third-party components offer their solutions for data synchronization, for example, **SharePlex** for Oracle by Quest Software, **WisdomForce** Database Sync System, **SymmetricDS** by symmetricds.codehaus.org. We don't describe benefits and shortcomings of these solutions in our article. Please contact these companies for more information about their respective product offerings.

**Conclusion**  
As we can see, the solutions mentioned in this article solve the task described in the beginning of it to some extent, but they have such limitations as difficulties in configuration, absence of a convenient GUI for fast analysis of differences, or high cost, or unavailable support for some Oracle versions.

### What Devart Offers

dbForge Data Compare for Oracle is a free [Oracle DB tool](https://www.devart.com/dbforge/oracle/) for Windows that simplifies table synchronization tasks. As a reliable [Oracle data compare](https://www.devart.com/dbforge/oracle/datacompare/) solution, it helps you visually identify differences between table records and synchronize them efficiently, whether you're working across development, testing, or production environments. It's easy to install and does not require any additional server or environment configuration, it supports all Oracle versions from 7.3 and 8i to Oracle XE and 11gR2. The comparison tool does not require any additional expenses for buying license, because it's totally free for commercial and non-commercial usage. Of course, it does not intend to replace replication completely, and its main task is searching for differences for further analysis and for their synchronization, if needed, and for creating script of CRUD operations for its further manual execution.

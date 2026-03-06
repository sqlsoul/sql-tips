---
layout: post
title: "Enabling AUTO_CLOSE Is a Bad Idea?"
description: "From a personal perspective, allowing a production database to run with AUTO_CLOSE option is not the best practice."
tags:
  - SQL Server
---

From a personal perspective, allowing a production database to run with *AUTO\_CLOSE* option is not the best practice. Let me explain why you should not enable *AUTO\_CLOSE* and the consequences of using this option.

The other day, I had to look in *Error Log* on a test server. After a two-minute timeout, I saw a great number of messages stored in the log, and I decided to check the log size using *xp\_enumerrorlogs*:

```sql
DECLARE @t TABLE (lod_id INT PRIMARY KEY, last_log SMALLDATETIME, size INT)
INSERT INTO @t
EXEC sys.xp_enumerrorlogs

SELECT lod_id, last_log, size_mb = size / 1048576.
FROM @t
```

```
lod_id   last_log              size_mb
-------- --------------------- ---------------
0        2016-01-05 08:46:00   567.05288505
1        2015-12-31 12:53:00   1370.39249420
2        2015-12-18 11:32:00   768.46394729
3        2015-12-02 13:54:00   220.20050621
4        2015-12-02 13:16:00   24.04152870
5        2015-11-16 13:37:00   80.07946205
6        2015-10-22 12:13:00   109.33527946
```

As usual, on test servers, I don't bother with the *Error Log* size, because each start of *SQL Server* initiates cyclic change of log files: the current *errorlog* is renamed to *errorlog.1*, an empty file *errorlog* is created and the earliest *errorlog.6* is deleted.

When I need to clean the logs, *sp\_cycle\_errorlog* can be helpful. But before cleaning the logs, it fell into my mind to see what interesting staff is recorded there.

I've read the current log with the stored procedure *xp\_readerrorlog*:

```sql
EXEC sys.xp_readerrorlog
```

And then I saw dozens of messages of this type:

```
Starting up database '...'.
```

On the one hand, there is nothing wrong with that. At each start, *SQL Server* opens data files and checks the boot page:

```
Starting up database '...'.
CHECKDB for database '...' finished without errors on ... (local time).
```

But after I have filtered by the message of interest, the results made me curious:

```sql
DECLARE @t TABLE (log_date SMALLDATETIME, spid VARCHAR(50), msg NVARCHAR(4000))
INSERT INTO @t
EXEC sys.xp_readerrorlog 0, 1, N'Starting up database'

SELECT msg, COUNT_BIG(1)
FROM @t
GROUP BY msg
HAVING COUNT_BIG(1) > 1
ORDER BY 2 DESC
```

```
------------------------------------------------------ --------------------
Starting up database 'AUTOTEST_DESCRIBER'.             127723
Starting up database 'MANUAL_DESCRIBER'.               12913
Starting up database 'AdventureWorks2012'.             12901
```

A great number of such messages may result from the *AUTO\_CLOSE* option set to ON.

According to the online documentation, when you turn on the *AUTO\_CLOSE* option, a database is shut down automatically and flush all resources after the last user logs off. When a new connection is requested, the database will automatically reopen…and so on ad infinitum.

Some time ago, I've read that in earlier versions of *SQL Server*, *AUTO\_CLOSE* was a synchronous process, which could cause long delays at repeated opening and closing of database files. Starting in *SQL Server 2005*, the *AUTO\_CLOSE* process became asynchronous, and partially the problem is gone now. But there are many issues with *AUTO\_CLOSE* that still remain.

To optimize performance, *SQL Server* changes pages in the buffer cache and does not write these pages to disk after each modification. Instead, *SQL Server* creates a checkpoint, at which it writes current pages modified in the memory, along with transaction log information from the memory to disk. When a database is shut down, *CHECKPOINT* is automatically executed. Accordingly, the disk load may greatly increase with the repeated database shutdowns.

Moreover, each database shutdown flushes the procedure cache. So, when the database reopens, the execution plans will have to be generated over again. But what is even worse, at the shutdown, the buffer cache also flushes, which increases disk load upon running queries.

What does *Microsoft* thinks about *AUTO\_CLOSE*?

*"When AUTO\_CLOSE is set ON, this option can cause performance degradation on frequently accessed databases because of the increased overhead of opening and closing the database after each connection. AUTO\_CLOSE also flushes the procedure cache after each connection"*

However, there are couple of nuances. In *SQL Server 2000* or any *Express* edition, when you create a new database, the *AUTO\_CLOSE* option will be enabled by default:

```sql
USE [master]
GO

IF DB_ID('test') IS NOT NULL
    DROP DATABASE [test]
GO

CREATE DATABASE [test]
GO

SELECT is_auto_close_on
FROM sys.databases
WHERE database_id = DB_ID('test')
```

```
is_auto_close_on
----------------
1
```

However, if you look at the upside, such *SQL Server Express* behavior is easy to explain, because this version sets a limit on the size of *RAM* usage – a maximum of 1 GB.

But in future, if you will need to deploy a database using a script, it is better to be on the safe side and explicitly disable *AUTO\_CLOSE*:

```sql
ALTER DATABASE [test] SET AUTO_CLOSE OFF
```

In the course of work, I've noticed one other interesting thing – when calling certain system functions or views, all databases with enabled *AUTO\_CLOSE* options will open:

```sql
USE [master]
GO

IF DB_ID('p1') IS NOT NULL
    DROP DATABASE [p1]
GO
CREATE DATABASE [p1]
GO
ALTER DATABASE [p1] SET AUTO_CLOSE ON
GO

IF DB_ID('p2') IS NOT NULL
    DROP DATABASE [p2]
GO
CREATE DATABASE [p2]
GO
ALTER DATABASE [p2] SET AUTO_CLOSE ON
GO

EXEC sys.xp_readerrorlog 0, 1, N'Starting up database ''p'
GO
```

```
LogDate                 ProcessInfo  Text
----------------------- ------------ ----------------------------------
2016-01-25 17:36:40.310 spid53       Starting up database 'p1'.
2016-01-25 17:36:41.980 spid53       Starting up database 'p2'.
```

We call *p1*:

```sql
WAITFOR DELAY '00:03'
GO
SELECT DB_ID('p1')
GO
EXEC sys.xp_readerrorlog 0, 1, N'Starting up database ''p'
```

But *p2* "wakes up" as well:

```
LogDate                 ProcessInfo  Text
----------------------- ------------ ----------------------------------
2016-01-25 17:36:40.310 spid53       Starting up database 'p1'.
2016-01-25 17:36:41.980 spid53       Starting up database 'p2'.
2016-01-25 17:39:17.440 spid52       Starting up database 'p1'.
2016-01-25 17:39:17.550 spid52       Starting up database 'p2'.
```

And finally we get to the main point. On a server, different users actively accessed metadata, making databases with enabled *AUTO\_CLOSE* open, which, in turn, caused the *Error Log* growth.

Preventive measures, by the way, are very simple:

```sql
DECLARE @SQL NVARCHAR(MAX)

SELECT @SQL = (
    SELECT '
ALTER DATABASE ' + QUOTENAME(name) + ' SET AUTO_CLOSE OFF WITH NO_WAIT;'
FROM sys.databases
WHERE is_auto_close_on = 1
    FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)')

EXEC sys.sp_executesql @SQL
```

All tests were implemented on *Microsoft SQL Server 2012 (SP3) (KB3072779) – 11.0.6020.0 (X64)*.

**Conclusion**

It may seem logical to close database that isn't in use to release resources and improve performance. But, in fact, it's harming your database far more than helping. So, unless you are absolutely sure that this feature is essential for you, the best practice is to leave the *AUTO\_CLOSE* setting *OFF*.

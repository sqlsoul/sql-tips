---
layout: post
title: "How to reduce MSDB size from 42Gb to 200Mb"
description: "How to troubleshoot an oversized SQL Server msdb database and reduce it from 42GB to 200MB by cleaning Service Broker queues, Database Mail data, and SQL Agent history."
tags:
  - performance
  - SQL Server
  - MSDB
---

Recently I've got a spare minute to see why an old test server was running too slow… I had nothing to do with this, but I was very anxious to find out what was wrong with the server.

First thing, I opened Resource Monitor and looked at the overall load. The *sqlserv.exe* process took up 100% of *CPU* and generated a large disk queue exceeding 300… whereas the number greater than 1 is considered problematic.

When analyzing disk activity, I observed continuous *IO* operations in *msdb*:

```
D:\SQL_2012\SYSTEM\MSDBData.mdf
D:\SQL_2012\SYSTEM\MSDBLog.ldf
```

I looked at the size of *msdb*:

```sql
SELECT name, size = size * 8. / 1024, space_used = FILEPROPERTY(name, 'SpaceUsed') * 8. / 1024
FROM sys.database_files
```

and switch to the "facepalm" mode:

```
name         size           space_used
------------ -------------- ---------------
MSDBData     42626.000000   42410.374395
MSDBLog      459.125000     6.859375
```

The data file takes up 42 GB… After a small break, I began to investigate the reason for such «unhealthy» size of *msdb*, and how to overcome the problems with server performance.

I've checked resource-cost queries running on the server:

```sql
SELECT
      r.session_id
    , db = DB_NAME(r.database_id)
    , r.[status]
    , p.[text]
    --, sql_text = SUBSTRING(p.[text], (r.statement_start_offset / 2) + 1,
    --        CASE WHEN r.statement_end_offset = -1
    --            THEN 2147483647
    --            ELSE ((r.statement_end_offset - r.statement_start_offset) / 2) + 1
    --        END)
    , r.cpu_time
    , r.total_elapsed_time
    , r.reads
    , r.writes
    , r.logical_reads
FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.[sql_handle]) p
WHERE r.[sql_handle] IS NOT NULL
    AND r.session_id != @@SPID
ORDER BY logical_reads DESC
```

The system stored procedure comes first:

```
session_id db       status   text                                  cpu_time    total_elapsed_time reads   writes    logical_reads
---------- -------- -------- ------------------------------------- ----------- ------------------ ------- --------- ---------------
62         msdb     running  create procedure [sys].[sp_cdc_scan]  111638      6739344            618232  554324    2857923422
```

I am referring to *CDC* (*Change Data Capture*), which is used as a tool for capturing data changes. *CDC* is based on reading the transaction log and always works asynchronously through the use of *Service Broker*.

Upon sending *Event Notification* to *Service Broker*, the message may not reach the destination due to configuration problems, and then it is archived in a separate table. In general, if *Service Broker* is frequently used, you need to monitor state of *sys.sysxmitqueue*. When there is constant increase of data in the table, it is either a bug or we use *Service Broker* incorrectly.

Execution plan from [dbForge Studio for SQL Server](https://www.devart.com/dbforge/sql/studio/sql-query-profiler.html):

![The execution plan from dbForge Studio for SQL Server](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/How-to-reduce-%20MSDB-size.png)

This query returns the top list of objects and their size:

```sql
USE msdb
GO

SELECT TOP(10)
      o.[object_id]
    , obj = SCHEMA_NAME(o.[schema_id]) + '.' + o.name
    , o.[type]
    , i.total_rows
    , i.total_size
FROM sys.objects o
JOIN (
    SELECT
          i.[object_id]
        , total_size = CAST(SUM(a.total_pages) * 8. / 1024 AS DECIMAL(18,2))
        , total_rows = SUM(CASE WHEN i.index_id IN (0, 1) AND a.[type] = 1 THEN p.[rows] END)
    FROM sys.indexes i
    JOIN sys.partitions p ON i.[object_id] = p.[object_id] AND i.index_id = p.index_id
    JOIN sys.allocation_units a ON p.[partition_id] = a.container_id
    WHERE i.is_disabled = 0
        AND i.is_hypothetical = 0
    GROUP BY i.[object_id]
) i ON o.[object_id] = i.[object_id]
WHERE o.[type] IN ('V', 'U', 'S')
ORDER BY i.total_size DESC
```

After running the query, I obtained the following results:

```
object_id   obj                               type total_rows   total_size 
----------- --------------------------------  ---- ------------ -----------
68          sys.sysxmitqueue                  S    6543502968   37188.90
942626401   dbo.sysmail_attachments           U    70           2566.00
1262627541  dbo.sysmail_attachments_transfer  U    35           2131.01
1102626971  dbo.sysmail_log                   U    44652        180.35
670625432   dbo.sysmail_mailitems             U    19231        123.39
965578478   dbo.sysjobhistory                 U    21055        69.05
366624349   dbo.backupfile                    U    6529         14.09 
727673640   dbo.sysssispackages               U    9            2.98  
206623779   dbo.backupset                     U    518          1.88  
286624064   dbo.backupfilegroup               U    3011         1.84
```

I must say that we will not leave all the tables in this list without attention. But first we need to fix issue with [sys.sysxmitqueue](https://docs.microsoft.com/en-us/previous-versions/sql/sql-server-2008/dd576261(v=sql.100)).

We cannot delete data directly from *sys.sysxmitqueue*, because this table is a system object *(S)*. After some searching, I found a way how to get *SQL Server* to clear the table. When creating a new *Service Broker*, all the messages associated with the old broker will be deleted.

```sql
USE msdb
GO

ALTER DATABASE msdb SET NEW_BROKER WITH ROLLBACK IMMEDIATE
```

But before executing the command, it is strongly recommended to disable *SQL Server Agent* and switch *SQL Server* to *Single-User Mode*. It took me approximately 10 minutes to delete the existing messages in all queues of *Service Broker*. On completion, I received the following message:

```
Nonqualified transactions are being rolled back. Estimated rollback completion: 100%.
```

After restarting the *SQL Server* service all performance problems were gone … the heart filled with joy and we could put a period here. But we should remember that it was not the only large table in *msdb*. Let's look at the rest…

Those who like to send mail via *Database Mail* should know that *SQL Server* logs and keep all the mailing in *msdb*. All e-mail attachments that are sent with the letter body are neatly stored there… Therefore, it is recommended to regularly delete this information. This can be done by hands, that is, look out for tables that need to be cleaned:

```sql
SELECT o.name, p.[rows]
FROM msdb.sys.objects o
JOIN msdb.sys.partitions p ON o.[object_id] = p.[object_id]
WHERE o.name LIKE 'sysmail%'
    AND o.[type] = 'U'
    AND p.[rows] > 0
```

Alternatively, use ready-to-use stored procedures [sysmail_delete_mailitems_sp](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sysmail-delete-mailitems-sp-transact-sql?view=sql-server-2017) and [sysmail_delete_log_sp](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sysmail-delete-log-sp-transact-sql?view=sql-server-2017):

```sql
DECLARE @DateBefore DATETIME 
SET @DateBefore = DATEADD(DAY, -7, GETDATE())

EXEC msdb.dbo.sysmail_delete_mailitems_sp @sent_before = @DateBefore --, @sent_status = 'sent'
EXEC msdb.dbo.sysmail_delete_log_sp @logged_before = @DateBefore
```

The history of *SQL Server Agent* tasks is also stored in *msdb*. When there are too many entries in the log, they are hard to work with, so I try to clean it regularly with [sp_purge_jobhistory](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-purge-jobhistory-transact-sql?view=sql-server-2017):

```sql
DECLARE @DateBefore DATETIME 
SET @DateBefore = DATEADD(DAY, -7, GETDATE())

EXEC msdb.dbo.sp_purge_jobhistory @oldest_date = @DateBefore
```

I should also mention about the information on backups that are logged in *msdb*. The old backup records can be deleted with [sp_delete_backuphistory](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-delete-backuphistory-transact-sql?view=sql-server-2017):

```sql
DECLARE @DateBefore DATETIME 
SET @DateBefore = DATEADD(DAY, -120, GETDATE())

EXEC msdb.dbo.sp_delete_backuphistory @oldest_date = @DateBefore
```

But we should remember about one nuance – when you delete a database, its backup info remain in *msdb*:

```sql
USE [master]
GO

IF DB_ID('backup_test') IS NOT NULL BEGIN
    ALTER DATABASE [backup_test] SET SINGLE_USER WITH ROLLBACK IMMEDIATE
    DROP DATABASE [backup_test]
END
GO

CREATE DATABASE [backup_test]
GO

BACKUP DATABASE [backup_test] TO DISK = N'backup_test.bak'
GO

DROP DATABASE [backup_test]
GO

SELECT *
FROM msdb.dbo.backupset
WHERE database_name = 'backup_test'
```

In my case, when databases are frequently created and deleted, it can lead to the increase of *msdb*. In the situation where backup information is of no use, it can be removed by the stored [sp_delete_database_backuphistory](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-delete-database-backuphistory-transact-sql?view=sql-server-2017):

```sql
EXEC msdb.dbo.sp_delete_database_backuphistory @database_name = N'backup_test'
```

## Summary

The system database *msdb* is used by many components of *SQL Server*, such as *Service Broker*, *SQL Server Agent*, and *Database Mail*. It is worth noting that there is no ready-for-use service plan, which would take into account the above-mentioned, so it is important to perform regular preventive measures. In my case, after deleting the unnecessary information and shrink the file, the size of *msdb* became 200 MB against the original 42 GB.

I hope this post will make an instructive story about the benefits of a permanent administration of both user and system databases.

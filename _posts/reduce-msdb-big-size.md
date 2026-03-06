---
layout: post
title: "How to reduce MSDB size from 42Gb to 200Mb"
description: "How to troubleshoot an oversized SQL Server msdb database and reduce it from 42GB to 200MB by cleaning Service Broker queues, Database Mail data, and SQL Agent history."
tags: performance, SQL Server
---

Recently I had a spare minute to investigate why an old test server was running too slow. First thing — I opened Resource Monitor and looked at the overall load. The `sqlserv.exe` process took up 100% of CPU and generated a large disk queue exceeding 300, whereas any value greater than 1 is already considered problematic.

When analyzing disk activity, I observed continuous IO operations in `msdb`:

```
D:\SQL_2012\SYSTEM\MSDBData.mdf
D:\SQL_2012\SYSTEM\MSDBLog.ldf
```

## Step 1: Check MSDB Size

I looked at the size of `msdb`:

```sql
SELECT name, size = size * 8. / 1024, space_used = FILEPROPERTY(name, 'SpaceUsed') * 8. / 1024
FROM sys.database_files
```

Result:

```
name         size           space_used
------------ -------------- ---------------
MSDBData     42626.000000   42410.374395
MSDBLog      459.125000     6.859375
```

The data file takes up **42 GB**. After a small pause, I began to investigate the root cause of this "unhealthy" size and how to fix the performance problems.

## Step 2: Find Resource-Intensive Queries

I checked which queries were consuming the most resources on the server:

```sql
SELECT
      r.session_id
    , db = DB_NAME(r.database_id)
    , r.[status]
    , p.[text]
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

The system stored procedure came first:

```
session_id  db    status   text                                  cpu_time  total_elapsed_time  reads   writes   logical_reads
----------  ----  -------  ------------------------------------  --------  ------------------  ------  -------  -------------
62          msdb  running  create procedure [sys].[sp_cdc_scan]  111638    6739344             618232  554324   2857923422
```

This pointed to **CDC (Change Data Capture)** — a tool for capturing data changes. CDC is based on reading the transaction log and always works asynchronously through **Service Broker**.

Upon sending an Event Notification to Service Broker, the message may not reach its destination due to configuration problems, and then gets archived in a separate table. If Service Broker is frequently used, you need to monitor the state of `sys.sysxmitqueue`. Constant data growth in that table indicates either a bug or incorrect Service Broker usage.

## Step 3: Find the Largest Tables in MSDB

This query returns the top objects and their sizes:

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

Result:

```
object_id   obj                               type  total_rows    total_size
-----------  --------------------------------  ----  ------------  -----------
68           sys.sysxmitqueue                  S     6543502968    37188.90
942626401    dbo.sysmail_attachments           U     70            2566.00
1262627541   dbo.sysmail_attachments_transfer  U     35            2131.01
1102626971   dbo.sysmail_log                   U     44652         180.35
670625432    dbo.sysmail_mailitems             U     19231         123.39
965578478    dbo.sysjobhistory                 U     21055         69.05
366624349    dbo.backupfile                    U     6529          14.09
727673640    dbo.sysssispackages               U     9             2.98
206623779    dbo.backupset                     U     518           1.88
286624064    dbo.backupfilegroup               U     3011          1.84
```

## Step 4: Fix sys.sysxmitqueue (Service Broker)

`sys.sysxmitqueue` alone takes up **~37 GB**. You cannot delete data from it directly since it's a system object. The way to clear it is to create a new Service Broker — this causes SQL Server to delete all messages associated with the old broker.

> ⚠️ **Important:** Before executing the command below, it is strongly recommended to **disable SQL Server Agent** and switch SQL Server to **Single-User Mode**.

```sql
USE msdb
GO

ALTER DATABASE msdb SET NEW_BROKER WITH ROLLBACK IMMEDIATE
```

This took approximately 10 minutes to delete all existing messages in the Service Broker queues. On completion, the following message was shown:

```
Nonqualified transactions are being rolled back. Estimated rollback completion: 100%.
```

After restarting the SQL Server service, all performance problems were gone.

## Step 5: Clean Up Database Mail Tables

SQL Server logs and stores all email activity in `msdb` — including attachments. It is recommended to regularly delete this data. First, check which tables need cleaning:

```sql
SELECT o.name, p.[rows]
FROM msdb.sys.objects o
JOIN msdb.sys.partitions p ON o.[object_id] = p.[object_id]
WHERE o.name LIKE 'sysmail%'
    AND o.[type] = 'U'
    AND p.[rows] > 0
```

Then use the built-in stored procedures to clean up:

```sql
DECLARE @DateBefore DATETIME 
SET @DateBefore = DATEADD(DAY, -7, GETDATE())

EXEC msdb.dbo.sysmail_delete_mailitems_sp @sent_before = @DateBefore
EXEC msdb.dbo.sysmail_delete_log_sp @logged_before = @DateBefore
```

## Step 6: Clean Up SQL Server Agent Job History

The history of SQL Server Agent jobs is also stored in `msdb`. Clean it regularly with `sp_purge_jobhistory`:

```sql
DECLARE @DateBefore DATETIME 
SET @DateBefore = DATEADD(DAY, -7, GETDATE())

EXEC msdb.dbo.sp_purge_jobhistory @oldest_date = @DateBefore
```

## Step 7: Clean Up Backup History

Old backup records can be deleted with `sp_delete_backuphistory`:

```sql
DECLARE @DateBefore DATETIME 
SET @DateBefore = DATEADD(DAY, -120, GETDATE())

EXEC msdb.dbo.sp_delete_backuphistory @oldest_date = @DateBefore
```

### Note on Deleted Databases

When a database is dropped, its backup history remains in `msdb`. This can be demonstrated with the following example:

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

In environments where databases are frequently created and deleted, this can lead to significant `msdb` growth. When backup history for a deleted database is no longer needed, remove it with:

```sql
EXEC msdb.dbo.sp_delete_database_backuphistory @database_name = N'backup_test'
```

## Summary

The system database `msdb` is used by many SQL Server components — **Service Broker**, **SQL Server Agent**, and **Database Mail**. There is no built-in maintenance plan that covers all of this automatically, so regular preventive cleanup is essential.

In my case, after deleting unnecessary data and shrinking the file, the size of `msdb` went from **42 GB down to 200 MB**.

| Cleanup Area | Stored Procedure |
|---|---|
| Service Broker queue | `ALTER DATABASE msdb SET NEW_BROKER` |
| Database Mail items | `msdb.dbo.sysmail_delete_mailitems_sp` |
| Database Mail log | `msdb.dbo.sysmail_delete_log_sp` |
| Agent job history | `msdb.dbo.sp_purge_jobhistory` |
| Backup history | `msdb.dbo.sp_delete_backuphistory` |
| Deleted DB backup history | `msdb.dbo.sp_delete_database_backuphistory` |

Regular administration of both user and system databases is just as important as maintaining your own databases — don't neglect `msdb`!

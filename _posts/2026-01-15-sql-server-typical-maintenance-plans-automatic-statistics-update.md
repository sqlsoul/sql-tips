---
layout: post
title: "SQL Server Typical Maintenance Plans: Automatic Statistics Update"
description: "The article demonstrates how to automatically update outdated statistics and statistics for large tables to increase query performance."
tags:
  - index fragmentation
  - performance
  - SQL Server
---

Some time ago, we reviewed the [automation of index defrag](https://blog.devart.com/sql-server-typical-maintenance-plans-part-1-automated-index-defragmentation.html). Now, it is time to look at statistics.

First of all, what do we need statistics for?

During execution of any query, query optimizer is trying to build an optimal execution plan (within the limits of available information). The plan constitutes the operations sequence, by means of which you can get the result described in the query.

While selecting one or another operation, the query optimizer considers statistics, that describes value distribution for columns within a table or index, as one of the most significant input data resources.

Such evaluation allows query optimizer to create more effective query plans. At the same time, if statistics contains outdated data, the less effective operations may be selected, that may result in slow execution plans. For instance, for a small selection based on outdated statistics, the more consuming Index Scan statement is selected instead of Index Seek.

As you see, statistics must be exact and fresh to be effective for query optimizer. Sometimes, SQL Server updates statistics on its own – this behavior is regulated by the AUTO\_CREATE\_STATISTICS and AUTO\_UPDATE\_STATISTICS options.

Besides, during index rebuild, their statistics is updated automatically with selection of the FULLSCAN option, that guarantees the most exact data distribution. On the contrary, statistics update does not take place during index reorganization.

If table data is modified as often as not, it is worth performing a selective statistics update manually with help of the UPDATE STATISTICS operation.

The manual update is also quite important when the NORECOMPUTE option is selected for statistics. The option means that the automatic statistics update is not required hereafter. You can view this property (as well as other properties) in the statistics properties:

```sql
SELECT s.*
FROM sys.stats s
JOIN sys.objects o ON s.[object_id] = o.[object_id]
WHERE o.is_ms_shipped = 0
```

Using possibilities of dynamic SQL, let's write a script for automatic update of outdated statistics:

```sql
DECLARE @DateNow DATETIME
SELECT @DateNow = DATEADD(dd, 0, DATEDIFF(dd, 0, GETDATE()))

DECLARE @SQL NVARCHAR(MAX)
SELECT @SQL = (
    SELECT '
        UPDATE STATISTICS [' + SCHEMA_NAME(o.[schema_id]) + '].[' + o.name + '] [' + s.name + ']
            WITH FULLSCAN' + CASE WHEN s.no_recompute = 1 THEN ', NORECOMPUTE' ELSE '' END + ';'
        FROM sys.stats s WITH(NOLOCK)
        JOIN sys.objects o WITH(NOLOCK) ON s.[object_id] = o.[object_id]
        WHERE o.[type] IN ('U', 'V')
            AND o.is_ms_shipped = 0
            AND ISNULL(STATS_DATE(s.[object_id], s.stats_id), GETDATE()) <= @DateNow
    FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)')

PRINT @SQL
EXEC sys.sp_executesql @SQL
```

While executing the script, the following statements will be generated:

```sql
UPDATE STATISTICS [Production].[Shift] [PK_Shift_ShiftID] WITH FULLSCAN;
UPDATE STATISTICS [Production].[Shift] [AK_Shift_Name] WITH FULLSCAN, NORECOMPUTE;
```

The statistics obsolescence criterion depends on the situation. In our example, it is 1 day.

In some cases, an excessive statistics update for large tables may drastically decrease database productivity. That's why, the given script is modifiable. For example, statistics for large tables can be updated less frequently:

```sql
DECLARE @DateNow DATETIME
SELECT @DateNow = DATEADD(dd, 0, DATEDIFF(dd, 0, GETDATE()))

DECLARE @SQL NVARCHAR(MAX)
SELECT @SQL = (
    SELECT '
        UPDATE STATISTICS [' + SCHEMA_NAME(o.[schema_id]) + '].[' + o.name + '] [' + s.name + ']
            WITH FULLSCAN' + CASE WHEN s.no_recompute = 1 THEN ', NORECOMPUTE' ELSE '' END + ';'
        FROM (
            SELECT
                  [object_id]
                , name
                , stats_id
                , no_recompute
                , last_update = STATS_DATE([object_id], stats_id)
            FROM sys.stats WITH(NOLOCK)
            WHERE auto_created = 0
                AND is_temporary = 0
        ) s
        JOIN sys.objects o WITH(NOLOCK) ON s.[object_id] = o.[object_id]
        JOIN (
            SELECT p.[object_id], p.index_id, total_pages = SUM(a.total_pages)
            FROM sys.partitions p WITH(NOLOCK)
            JOIN sys.allocation_units a WITH(NOLOCK) ON p.[partition_id] = a.container_id
            GROUP BY p.[object_id], p.index_id
        ) p ON o.[object_id] = p.[object_id] AND p.index_id = s.stats_id
        WHERE o.[type] IN ('U', 'V')
            AND o.is_ms_shipped = 0
            AND (
                    last_update IS NULL AND p.total_pages > 0 -- never updated and contains rows
                OR
                    last_update < DATEADD(dd,
                        CASE
                            WHEN p.total_pages > 4096 -- > 4 MB
                            THEN -2 -- updated 3 days ago
                            ELSE 0
                        END, @DateNow)
            )
    FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)')

PRINT @SQL
EXEC sys.sp_executesql @SQL
```

In the next post, we will consider database backup automation.

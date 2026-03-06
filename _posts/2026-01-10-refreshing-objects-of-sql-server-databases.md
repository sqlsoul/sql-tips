---
layout: post
title: "Refreshing Objects of SQL Server Databases"
description: "Learn how to perform database refreshes in SQL Server using manual restore, automated scripts and importance of metadata update with sp_refreshsqlmodule"
tags:
  - SQL Server
---

In some cases, for example, when migrating or updating objects, it's necessary **to update metadata** for views created without evident listing of columns in the select-list, i. e. for views of the SELECT \* FROM some\_table type. The built-in **sp\_refreshview** procedure is developed for these goals, it updates the metadata for the specified *non-schema-bound view*. Persistent metadata for a view can become outdated because of changes to the underlying objects upon which the view depends.

Metadata of other objects containing the code can be changed using the built-in **sp\_refreshsqlmodule** procedure, that is designed to update metadata for the specified *non-schema-bound stored procedure*, *user-defined function*, *view*, *DML trigger* or *database-level DDL trigger* in the current database. Persistent metadata for these objects, such as data types of parameters, can become outdated because of changes to their underlying objects.

So using the script specified below, the updates of all objects containing the code for the current database can be performed (i.e. update metadata information for all VIEWS, DML TRIGGERS, PROCEDURES, FUNCTIONS, DDL TRIGGERS):

```sql
USE [your_db]
GO
PRINT ' -- Refreshing all VIEWS in database ' + QUOTENAME(DB_NAME()) + ' :'
DECLARE @stmt_refresh_object nvarchar(400)
DECLARE c_refresh_object CURSOR FOR
SELECT DISTINCT 'EXEC sp_refreshview '''
+QUOTENAME(ss.name)+'.'
+QUOTENAME(so.name)+'''' as stmt_refresh_views
FROM sys.objects AS so 
   INNER JOIN sys.sql_expression_dependencies AS sed 
      ON so.object_id = sed.referencing_id 
   INNER JOIN sys.schemas AS ss
      ON so.schema_id = ss.schema_id
WHERE so.type = 'V' AND sed.is_schema_bound_reference = 0
OPEN c_refresh_object
FETCH NEXT FROM c_refresh_object INTO @stmt_refresh_object
WHILE @@FETCH_STATUS = 0
   BEGIN
       print @stmt_refresh_object
       exec sp_executesql @stmt_refresh_object
      FETCH NEXT FROM c_refresh_object INTO @stmt_refresh_object
   END
CLOSE c_refresh_object
DEALLOCATE c_refresh_object
GO
PRINT ' -- Refreshing all DML TRIGGERS in database ' + QUOTENAME(DB_NAME()) + ' :'
DECLARE @stmt_refresh_object nvarchar(400)
DECLARE c_refresh_object CURSOR FOR
SELECT DISTINCT 'EXEC sp_refreshsqlmodule '''
+QUOTENAME(schemas.name)+'.'
+QUOTENAME(triggers.name)+'''' 
as stmt_refresh_dml_triggers
FROM sys.triggers AS triggers WITH(NOLOCK)
  INNER JOIN sys.objects AS objects WITH(NOLOCK) 
     ON objects.object_id = triggers.parent_id
  INNER JOIN sys.schemas AS schemas WITH(NOLOCK) 
     ON schemas.schema_id = objects.schema_id
  LEFT JOIN sys.sql_modules AS sql_modules WITH(NOLOCK) 
     ON sql_modules.object_id = triggers.object_id
  LEFT JOIN sys.assembly_modules AS assembly_modules WITH(NOLOCK) 
     ON assembly_modules.object_id = triggers.object_id
  LEFT JOIN sys.assemblies AS assemblies WITH(NOLOCK) 
     ON assemblies.assembly_id = assembly_modules.assembly_id
  LEFT JOIN sys.database_principals AS principals WITH(NOLOCK) 
     ON principals.principal_id = assembly_modules.execute_as_principal_id 
       OR principals.principal_id = sql_modules.execute_as_principal_id
WHERE RTRIM(objects.type) IN ('U','V') and parent_class = 1 
    AND sql_modules.is_schema_bound = 0
OPEN c_refresh_object
FETCH NEXT FROM c_refresh_object INTO @stmt_refresh_object
WHILE @@FETCH_STATUS = 0
   BEGIN
       print @stmt_refresh_object
       exec sp_executesql @stmt_refresh_object
      FETCH NEXT FROM c_refresh_object INTO @stmt_refresh_object
   END
CLOSE c_refresh_object
DEALLOCATE c_refresh_object
GO
PRINT ' -- Refreshing all PROCEDURES in database ' + QUOTENAME(DB_NAME()) + ' :'
DECLARE @stmt_refresh_object nvarchar(400)
DECLARE c_refresh_object CURSOR FOR
SELECT DISTINCT 'EXEC sp_refreshsqlmodule '''
+QUOTENAME(s.name)+'.'
+QUOTENAME(p.name)+''''
as stmt_refresh_procedures
FROM
  sys.procedures AS p WITH(NOLOCK)
LEFT JOIN
  sys.schemas AS s WITH(NOLOCK)
     ON p.schema_id = s.schema_id
LEFT JOIN
  sys.sql_modules AS sm WITH(NOLOCK)
     ON p.object_id = sm.object_id
LEFT JOIN
  sys.assembly_modules AS am WITH(NOLOCK)
     ON p.object_id = am.object_id
LEFT JOIN
  sys.assemblies AS a
     ON a.assembly_id = am.assembly_id
LEFT JOIN
  sys.objects AS o WITH(NOLOCK)
     ON sm.object_id = o.object_id
LEFT JOIN
  sys.database_principals AS dp WITH(NOLOCK)
     ON sm.execute_as_principal_id = dp.principal_id 
        OR am.execute_as_principal_id = dp.principal_id
LEFT JOIN
  sys.database_principals AS dp1 WITH(NOLOCK)
     ON o.principal_id = dp1.principal_id
WHERE
  (CAST(CASE  WHEN p.is_ms_shipped = 1 then 1
            WHEN (SELECT major_id FROM sys.extended_properties WHERE
                  major_id = p.object_id AND minor_id = 0 AND class = 1 AND
                  name = 'microsoft_database_tools_support') IS NOT NULL THEN 1
            ELSE 0 END AS bit)=0)
OPEN c_refresh_object
FETCH NEXT FROM c_refresh_object INTO @stmt_refresh_object
WHILE @@FETCH_STATUS = 0
   BEGIN
       print @stmt_refresh_object
       exec sp_executesql @stmt_refresh_object
      FETCH NEXT FROM c_refresh_object INTO @stmt_refresh_object
   END
CLOSE c_refresh_object
DEALLOCATE c_refresh_object
GO
PRINT ' -- Refreshing all FUNCTIONS in database ' + QUOTENAME(DB_NAME()) + ' :'
DECLARE @stmt_refresh_object nvarchar(400)
DECLARE c_refresh_object CURSOR FOR
SELECT DISTINCT 'EXEC sp_refreshsqlmodule '''
+QUOTENAME(SCHEMA_NAME(o.schema_id))+'.'
+QUOTENAME(o.name)+'''' 
as stmt_refresh_functions
FROM sys.objects AS o WITH(NOLOCK)
  LEFT JOIN sys.sql_modules AS sm WITH(NOLOCK) 
      ON o.object_id = sm.object_id
  LEFT JOIN sys.assembly_modules AS am WITH(NOLOCK) 
      ON o.object_id = am.object_id
  LEFT JOIN sys.database_principals p1 WITH(NOLOCK) 
      ON p1.principal_id = o.principal_id
  LEFT JOIN sys.database_principals p2 WITH(NOLOCK) 
      ON p2.principal_id=am.execute_as_principal_id
  LEFT JOIN sys.database_principals p3 WITH(NOLOCK) 
      ON p3.principal_id=sm.execute_as_principal_id
  LEFT JOIN sys.assemblies AS ass WITH(NOLOCK) 
      ON ass.assembly_id = am.assembly_id
WHERE o.type IN ('FN','IF','TF','AF','FS','FT') and sm.is_schema_bound = 0
OPEN c_refresh_object
FETCH NEXT FROM c_refresh_object INTO @stmt_refresh_object
WHILE @@FETCH_STATUS = 0
   BEGIN
       print @stmt_refresh_object
       exec sp_executesql @stmt_refresh_object
      FETCH NEXT FROM c_refresh_object INTO @stmt_refresh_object
   END
CLOSE c_refresh_object
DEALLOCATE c_refresh_object
GO
PRINT ' -- Refreshing all DDL TRIGGERS on database ' + QUOTENAME(DB_NAME()) + ' :'
DECLARE @stmt_refresh_object nvarchar(400)
DECLARE c_refresh_object CURSOR FOR
SELECT DISTINCT 'EXEC sp_refreshsqlmodule '''
+QUOTENAME(t.name)+''','
+'''DATABASE_DDL_TRIGGER''' as stmt_refresh_ddl_triggers
FROM sys.triggers AS t WITH(NOLOCK)
  LEFT JOIN sys.sql_modules AS sm WITH(NOLOCK) 
     ON t.object_id = sm.object_id
  LEFT JOIN sys.assembly_modules AS am WITH(NOLOCK) 
     ON t.object_id = am.object_id
  LEFT JOIN sys.assemblies AS assemblies WITH(NOLOCK) 
     ON assemblies.assembly_id = am.assembly_id
  LEFT JOIN sys.database_principals AS principals WITH(NOLOCK) 
     ON principals.principal_id = sm.execute_as_principal_id 
        OR principals.principal_id = am.execute_as_principal_id
WHERE parent_class = 0
OPEN c_refresh_object
FETCH NEXT FROM c_refresh_object INTO @stmt_refresh_object
WHILE @@FETCH_STATUS = 0
   BEGIN
       print @stmt_refresh_object
       exec sp_executesql @stmt_refresh_object
      FETCH NEXT FROM c_refresh_object INTO @stmt_refresh_object
   END
CLOSE c_refresh_object
DEALLOCATE c_refresh_object
GO
PRINT 'Metadata update for non-schema-bound objects is done.'
GO
USE [master]
GO
```

Here the result in Microsoft SQL Server Management Studio:

![MSSMS](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/MSSMS.png)

MSSMS

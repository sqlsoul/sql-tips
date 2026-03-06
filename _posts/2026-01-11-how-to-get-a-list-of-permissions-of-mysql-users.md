---
layout: post
title: "How to Get a List of Permissions of MySQL Users"
description: "In this article Devart describes how you can create a list of privileges of MySQL database users with the help of a data reports tool."
tags:
  - MySQL
  - dbForge Studio for MySQL
---

MySQL has an advanced database access system. A database user can have access to the whole database, separate tables, or to separate columns of tables. Also, there is a restriction for actions a user may perform with records.

MySQL server uses several tables in a special database to organize such a complicated database access structure. The access policy is based on the values available in these tables.

The database that MySQL server uses to store internal data about users is called *mysql* by default. There are tables for storing information about users' accounts in this database:

* ***user*** contains a list of all users of the MySQL server and their permissions to access the database;
* ***db*** contains a list of databases with a matching list of database users and their privileges for executing operations;
* ***tables\_priv*** contains a list of database tables (views) that a user has access to;
* ***columns\_priv*** contains a list of columns from the database tables (views) a user has access to;
* ***procs\_priv*** contains a list of database procedures (functions) a user has access to.

To get the list of users' privileges concerning data access, the following queries may be executed:

* the list of global privileges:

```sql
  SELECT
  mu.host `Host`,
  mu.user `User`,
  REPLACE(RTRIM(CONCAT(
  IF(mu.Select_priv = 'Y', 'Select ', ''),
  IF(mu.Insert_priv = 'Y', 'Insert ', ''),
  IF(mu.Update_priv = 'Y', 'Update ', ''),
  IF(mu.Delete_priv = 'Y', 'Delete ', ''),
  IF(mu.Create_priv = 'Y', 'Create ', ''),
  IF(mu.Drop_priv = 'Y', 'Drop ', ''),
  IF(mu.Reload_priv = 'Y', 'Reload ', ''),
  IF(mu.Shutdown_priv = 'Y', 'Shutdown ', ''),
  IF(mu.Process_priv = 'Y', 'Process ', ''),
  IF(mu.File_priv = 'Y', 'File ', ''),
  IF(mu.Grant_priv = 'Y', 'Grant ', ''),
  IF(mu.References_priv = 'Y', 'References ', ''),
  IF(mu.Index_priv = 'Y', 'Index ', ''),
  IF(mu.Alter_priv = 'Y', 'Alter ', ''),
  IF(mu.Show_db_priv = 'Y', 'Show_db ', ''),
  IF(mu.Super_priv = 'Y', 'Super ', ''),
  IF(mu.Create_tmp_table_priv = 'Y', 'Create_tmp_table ', ''),
  IF(mu.Lock_tables_priv = 'Y', 'Lock_tables ', ''),
  IF(mu.Execute_priv = 'Y', 'Execute ', ''),
  IF(mu.Repl_slave_priv = 'Y', 'Repl_slave ', ''),
  IF(mu.Repl_client_priv = 'Y', 'Repl_client ', ''),
  IF(mu.Create_view_priv = 'Y', 'Create_view ', ''),
  IF(mu.Show_view_priv = 'Y', 'Show_view ', ''),
  IF(mu.Create_routine_priv = 'Y', 'Create_routine ', ''),
  IF(mu.Alter_routine_priv = 'Y', 'Alter_routine ', ''),
  IF(mu.Create_user_priv = 'Y', 'Create_user ', ''),
  IF(mu.Event_priv = 'Y', 'Event ', ''),
  IF(mu.Trigger_priv = 'Y', 'Trigger ', '')
  )), ' ', ', ') AS `Privileges`
 FROM
  mysql.user mu
 ORDER BY
  mu.Host,
  mu.User
```

* the list of privileges for a database:

```sql
 SELECT 
  md.host `Host`,
  md.user `User`,
  md.db `Database`,
  REPLACE(RTRIM(CONCAT(
  IF(md.Select_priv = 'Y', 'Select ', ''),
  IF(md.Insert_priv = 'Y', 'Insert ', ''),
  IF(md.Update_priv = 'Y', 'Update ', ''),
  IF(md.Delete_priv = 'Y', 'Delete ', ''),
  IF(md.Create_priv = 'Y', 'Create ', ''),
  IF(md.Drop_priv = 'Y', 'Drop ', ''),
  IF(md.Grant_priv = 'Y', 'Grant ', ''),
  IF(md.References_priv = 'Y', 'References ', ''),
  IF(md.Index_priv = 'Y', 'Index ', ''),
  IF(md.Alter_priv = 'Y', 'Alter ', ''),
  IF(md.Create_tmp_table_priv = 'Y', 'Create_tmp_table ', ''),
  IF(md.Lock_tables_priv = 'Y', 'Lock_tables ', ''),
  IF(md.Create_view_priv = 'Y', 'Create_view ', ''),
  IF(md.Show_view_priv = 'Y', 'Show_view ', ''),
  IF(md.Create_routine_priv = 'Y', 'Create_routine ', ''),
  IF(md.Alter_routine_priv = 'Y', 'Alter_routine ', ''),
  IF(md.Execute_priv = 'Y', 'Execute ', ''),
  IF(md.Event_priv = 'Y', 'Event ', ''),
  IF(md.Trigger_priv = 'Y', 'Trigger ', '')
  )), ' ', ', ') AS `Privileges`
 FROM
  mysql.db md
 ORDER BY
  md.Host,
  md.User,
  md.Db
```

* the list of privileges for tables:

```sql
 SELECT 
  mt.host `Host`,
  mt.user `User`,
  CONCAT(mt.Db, '.', mt.Table_name) `Tables`,
  REPLACE(mt.Table_priv, ',', ', ') AS `Privileges`
 FROM
  mysql.tables_priv mt
 WHERE
  mt.Table_name IN
  (SELECT
  DISTINCT
    t.table_name `tables`
  FROM
    information_schema.tables AS t
  WHERE
    t.table_type IN
    ('BASE TABLE', 'SYSTEM VIEW', 'TEMPORARY', '') OR
    t.table_type <> 'VIEW' AND
    t.create_options IS NOT NULL
  )
 ORDER BY
  mt.Host,
  mt.User,
  mt.Db,
  mt.Table_name;
```

* the list of privileges for views:

```sql
 SELECT 
  mv.host `Host`,
  mv.user `User`,
  CONCAT(mv.Db, '.', mv.Table_name) `Views`,
  REPLACE(mv.Table_priv, ',', ', ') AS `Privileges`
 FROM
  mysql.tables_priv mv
 WHERE
  mv.Table_name IN
  (SELECT
  DISTINCT
    v.table_name `views`
  FROM
    information_schema.views AS v
  )
 ORDER BY
  mv.Host,
  mv.User,
  mv.Db,
  mv.Table_name;
```

* the list of privileges for table columns:

```sql
SELECT 
  mtc.host `Host`,
  mtc.user `User`,
  CONCAT(mtc.Db, '.', mtc.Table_name, '.', mtc.Column_name) `Tables Columns`,
  REPLACE(mtc.Column_priv, ',', ', ') AS `Privileges`
FROM
  mysql.columns_priv mtc
WHERE
  mtc.Table_name IN
  (SELECT
  DISTINCT
    t.table_name `tables`
  FROM
    information_schema.tables AS t
  WHERE
    t.table_type IN
    ('BASE TABLE', 'SYSTEM VIEW', 'TEMPORARY', '') OR
    t.table_type <> 'VIEW' AND
    t.create_options IS NOT NULL
  )
ORDER BY
  mtc.Host,
  mtc.User,
  mtc.Db,
  mtc.Table_name,
  mtc.Column_name;
```

* the list of privileges for view columns:

```sql
 SELECT
  mvc.host `Host`,
  mvc.user `User`,
  CONCAT(mvc.Db, '.', mvc.Table_name, '.', mvc.Column_name) `Views Columns`,
  REPLACE(mvc.Column_priv, ',', ', ') AS `Privileges`
 FROM
  mysql.columns_priv mvc
 WHERE
  mvc.Table_name IN
  (SELECT
  DISTINCT
    v.table_name `views`
  FROM
    information_schema.views AS v
  )
 ORDER BY
  mvc.Host,
  mvc.User,
  mvc.Db,
  mvc.Table_name,
  mvc.Column_name;
```

* the list of privileges for procedures:

```sql
 SELECT
  mp.host `Host`,
  mp.user `User`,
  CONCAT(mp.Db, '.', mp.Routine_name) `Procedures`,
  REPLACE(mp.Proc_priv, ',', ', ') AS `Privileges`
 FROM
  mysql.procs_priv mp
 WHERE
  mp.Routine_type = 'PROCEDURE'
 ORDER BY
  mp.Host,
  mp.User,
  mp.Db,
  mp.Routine_name;
```

* the list of privileges for functions:

```sql
 SELECT
  mf.host `Host`,
  mf.user `User`,
  CONCAT(mf.Db, '.', mf.Routine_name) `Procedures`,
  REPLACE(mf.Proc_priv, ',', ', ') AS `Privileges`
 FROM
  mysql.procs_priv mf
 WHERE
  mf.Routine_type = 'FUNCTION'
 ORDER BY
  mf.Host,
  mf.User,
  mf.Db,
  mf.Routine_name;
```

You may need to create a decent printable report with this data and to give it as a report, for example, by the demand of a customer or authority. For this purpose, you may use a special [MySQL GUI tool](https://www.devart.com/dbforge/mysql/studio/) that includes a [data report designer](https://www.devart.com/dbforge/mysql/studio/data-reports.html).

If you have ready queries, you can take advantage of an easy-to-use wizard and create a report using a predefined template and data grouped, for example, by the host, in several minutes.

![New Data Report menu](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/01New_Data_Report_menu.png)

New Data Report menu

![Data Report Wizard](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/02Data_Report_Wizard.png)

Data Report Wizard

![Data Report Custom Query](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/03Data_Report_Custom_Query.png)

Data Report Custom Query

![Data Report Load Query](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/04Data_Report_Load_Query.png)

Data Report Load Query

![Data Report Group by Host](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/05Data_Report_Group_by_Host.png)
Data Report Group by Host

![Data Report Title](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/06Data_Report_Title.png)
Data Report Title

![Data Report Preview](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/07Data_Report_Preview.png)

Data Report Preview

As you can see on these screenshots, we have created a report using dbForge Studio's wizard without tedious designing the report itself.

## Conclusion

In the article, we have described with particular scripts how to get a list of users' privileges concerning data access.

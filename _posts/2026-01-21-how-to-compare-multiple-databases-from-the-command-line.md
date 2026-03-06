---
layout: post
title: "How to Automatically Synchronize Multiple Databases on Different SQL Server Instances"
description: "Read this tutorial to know how to automatically compare and synchronize multiple SQL Server databases located on different servers using dbForge Data and Schema Compare for SQL Server."
tags:
  - command line
  - compare multiple databases
  - data compare
  - SQL Server Tutorial
---

![](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/1068x580_how-to-compare-multiple-databases-from-the-command-line.png)

In this article, we share a step-by-step guide on how to compare the schema and data of multiple SQL Server databases from the command line.

Both [dbForge Schema Compare for SQL Server](https://www.devart.com/dbforge/sql/schemacompare/) and [dbForge Data Compare for SQL Server](https://www.devart.com/dbforge/sql/datacompare/) support the command-line interface, which gives the user a rich control over the tools and allows automating and scheduling regular database comparison and synchronization tasks.

As a DBA or SQL Server developer, you may face situations when you need to compare schema and/or data in more than two databases and quickly sync two or more SQL Server databases. Can you do it in one go and save a lot of time and effort? Let's check.

### Step 1. Create a text file with the list of source and target databases and servers

1.1 Launch any third-party text editor, for example, Notepad.

1.2 Enter the names of the source servers and databases, separated by commas. Here you can write as many servers and databases as you need. Below is the template for such a list:

```
Source_server_name1, Source_database_name1, Source_user1, Password1
Source_server_name2, Source_database_name2, Source_user2, Password2
Source_server_name3, Source_database_name3, Source_user2, Password3
...
Source_server_nameN, Source_database_nameN, Source_userN, PasswordN
```

In this worked example, we are going to use as Source the following databases on the following servers:

```
DBFSQLSRV\SQL2016, AdventureWorks2019_Dev, Source_user1, Password1
DBFSQLSRV\SQL2016, BicycleStoreDev, Source_user2, Password2
DBFSQLSRV\SQL2016, BicycleStoreDemo, Source_user3, Password3
```

1.3 Save the file. We will save the file with the name *source\_servers\_databases.txt*.

1.4 To create multitarget, repeat the previous step for the target servers and databases by entering their names separated by commas according to the template:

```
Target_server_name1, Target_database_name1, Target_user1, Password1
Target_server_name2, Target_database_name2, Target_user2, Password2
Target_server_name3, Target_database_name3, Target_user3, Password3
...
Target_server_nameN, Target_database_nameN, Target_userN, PasswordN
```

In this worked example, we are going to use as Target the following databases on the following servers:

```
DBFSQLSRV\SQL2019, AdventureWorks2019_Test, Target_user1, Password1
DBFSQLSRV\SQL2019, BicycleStoreDev, Target_user2, Password2
DBFSQLSRV\SQL2019, BicycleStoreDemo, Target_user3, Password3
```

1.5 Save the file. We will save the file with the name *target\_servers\_databases.txt*.

### Step 2. Create a .bat file

2.1 Launch any third-party text editor, for example, Notepad.

2.2 Enter the script for comparing databases like in the examples below. Don't forget to adjust the script to suit your needs.

**The script for comparing schemas of multiple databases from the command line:**

```bat
set StudioPath="C:\Program Files\Devart\dbForge Studio for SQL Server\dbforgesql.com"

set ToolComparePath="C:\Program Files\Devart\Compare Bundle for SQL Server\dbForge Schema Compare for SQL Server\schemacompare.com"

FOR /F "eol=; tokens=1,2,3,4* delims=, " %%e in (source_servers_databases.txt) do (

FOR /F "eol=; tokens=1,2,3,4* delims=, " %%i in (target_servers_databases.txt) do (

%ToolComparePath% /schemacompare /source connection:"Data Source=%%e;Encrypt=False;Enlist=False;Initial Catalog=%%f;Integrated Security=False;User ID=%%g;Password=%%h;Pooling=False;Transaction Scope Local=True" /target connection:"Data Source=%%i;Encrypt=False;Enlist=False;Initial Catalog=%%j;Integrated Security=False;User ID=%%k;Password=%%l;Pooling=False;Transaction Scope Local=True" /log:"schema_compare_sql_log.log" 

) 

)
pause
```

**The script for comparing data of multiple databases from the command line:**

```bat
set StudioPath="C:\Program Files\Devart\dbForge Studio for SQL Server\dbforgesql.com"

set ToolComparePath="C:\Program Files\Devart\Compare Bundle for SQL Server\dbForge Data Compare for SQL Server\datacompare.com"

FOR /F "eol=; tokens=1,2,3,4* delims=, " %%e in (source_servers_databases.txt) do (

FOR /F "eol=; tokens=1,2,3,4* delims=, " %%i in (target_servers_databases.txt) do (

%ToolComparePath% /datacompare /source connection:"Data Source=%%e;Encrypt=False;Enlist=False;Initial Catalog=%%f;Integrated Security=False;User ID=%%g;Password=%%h;Pooling=False;Transaction Scope Local=True" /target connection:"Data Source=%%i;Encrypt=False;Enlist=False;Initial Catalog=%%j;Integrated Security=False;User ID=%%k;Password=%%l;Pooling=False;Transaction Scope Local=True" /log:"data_compare_sql_log.log" 

) 

)
pause
```

**Where:**

* *source\_servers\_databases.txt* is the name of the file listing source servers and databases.
* *target\_servers\_databases.txt* is the name of the file listing source servers and databases.
* *%ToolComparePath%* identifies that you are using dbForge Schema Compare for SQL Server and dbForge Data Compare for SQL Server. If you want to use dbForge Studio for SQL Server, replace the variable with *%StudioPath%*
* */log: data\_compare\_sql\_log.log* is a path to the file where the output result will be stored.

Pay attention that each script ends with *pause*. The command is provided to prevent the command-line window from automatically closing. This is necessary for monitoring the execution progress. To use it in production, *pause* should be removed from the script.

*Note: ToolComparePath is a default installation path for dbForge Data Compare and dbForge Schema Compare. However, if you have changed it, you will need to specify the correct path to the required tool's .com file as well.*

2.3 Save the script.

### Step 3. Compare source and target databases via the command line

Now, all you need to do is execute the .bat file via the command line.

First, let us run the .bat file for comparing schemas in our set of databases.

![schema compare result](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/schema-compare-result-1.png)

Now, let us run the .bat file for comparing data in our databases.

![data compare result](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/data-compare-result-1.png)

You will instantly get the summary comparison results: whether the source and target databases are identical or not, how many different or conflicting records there are, etc. The output file will be generated after the successful completion of the process.

Additionally, if you want to learn how to automate and schedule SQL database synchronization from the CLI, feel free to watch [this video.](https://youtu.be/44kk1a-AGxY)

To learn more about comparing data and schemas from the command line, refer to the corresponding documentation topics:

* [Compare data in multiple databases from the command line](https://docs.devart.com/schema-compare-for-sql-server/working-with-particular-cases/compare-schemas-in-multiple-databases-from-the-command-line.html)
* [Compare schemas in multiple databases from the command line](https://docs.devart.com/schema-compare-for-sql-server/working-with-particular-cases/compare-schemas-in-multiple-databases-from-the-command-line.html)

To learn more about how to sync MySQL databases, refer to the [MySQL database synchronization](https://www.devart.com/dbforge/mysql/studio/database-synchronization.html) page.

### Conclusion

dbForge Schema Compare and dbForge Data Compare tools include the command-line interface (CLI) for performing schema comparison and deployments of SQL Server databases from the command line, thus allowing multi db synchronization. This article provides worked examples of CLI scripts for comparing SQL Server schemas and data across multiple databases. Try the given scenario and you will see how easy it is to sync two databases on different servers.

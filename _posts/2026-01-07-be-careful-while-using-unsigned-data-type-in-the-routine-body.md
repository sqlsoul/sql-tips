---
layout: post
title: "Be careful while using UNSIGNED data type in the routine body"
description: "MySQL stored procedure example showing why using the UNSIGNED data type in routine variables may cause silent value changes and unexpected results."
tags:
  - debugging
  - MySQL
  - dbForge Studio for MySQL
---

**Introduction**

MySQL Server (starting from v 5.0), as Oracle and SQL Servers, allows creating [stored procedures and functions](https://dev.mysql.com/doc/refman/8.0/en/stored-routines.html).

Stored procedures are a set of SQL commands that can be compiled and stored on the server. Thus instead of storing a frequently-used query, we can refer to a corresponding stored procedure. This provides better performance (as this query is analyzed only once) and reduction of traffic between client and server.

While developing business logic of procedures, we often use a great number of variables (e.g., temporary outputs) to store. To assign static values to a variable or values of other variables, SET operator is used. [SET operator in stored procedures](https://dev.mysql.com/doc/refman/8.0/en/stored-programs-defining.html) is an extended version of usual SET operator. This allows using extended syntax **SET а=х, Ь=у**, where different variables types (local and server variables, global and session ones) can be mixed.

**Problem**

While assigning a value which size exceeds the maximum data type size of a variable, MySQL Server should show "Data type size exceeded" error. In such situations farther execution of procedure's code with the error should be aborted.

Let us illustrate this case exploring how MySQL Server reacts when a variable value with [SMALLINT](https://dev.mysql.com/doc/refman/8.0/en/numeric-types.html) data type is assigned to a variable with [TINYINT](https://dev.mysql.com/doc/refman/8.0/en/numeric-types.html) data type.

![A variable value with SMALLINT data type is assigned to a variable with TINYINT data type](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/code_1.gif)

![MySQL threw Out of Range exception.](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/code_2.gif)

Here DECLARE v\_TINYINT TINYINT means that any value in the range -128 to 127 can be assign to the variable. If we assign a value being out of this range, for example, assign 250 to the variable with SMALLINT data type where the tolerable limit is -32768 to 32767 range, MySQL Server will show "[Out of range](https://dev.mysql.com/doc/refman/5.1/en/error-messages-server.html#error_er_warn_data_out_of_range)" error. A poor developer should review the procedure's code.

If the code contains UNSIGNED data type, the situation is a bit different. [UNSIGNED](https://dev.mysql.com/doc/refman/8.0/en/numeric-types.html) is a special instruction for main data type, extending a positive limit twice and excluding the negative limit.

To illustrate this case, let us assign a variable value with TINYINT UNSIGNED data type to a variable with TINYINT data type.

![Assign a variable value with TINYINT UNSIGNED data type to a variable with TINYINT data type](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/code_3.gif)

![MySQL did not throw any exception.](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/code_4.gif)

DECLARE v\_TINYINTUNSIGNED TINYINT UNSIGNED means that assigned values of v\_TINYINTUNSIGNED variable should be within 0 to 255 range. We assigned TINYINT UNSIGNED value that exceeded the maximum size – 127. After assigning, MySQL Server has stored value – 6, instead of expected – 250. No errors or warnings were shown. You will most likely pay no attention to this, but it can cause unexpected results after executing the procedure's code.

**Solution**

Here are our recommendations for such cases:

1. Watch over the correspondence between data types of variables during evaluation.
2. Do not use UNSIGNED at all, instead use a character type of a big size.
3. Use [visual tools for debugging stored procedures](https://www.devart.com/dbforge/mysql/studio/debugging.html) to find such errors. Try **dbForge Studio for MySQL**, it has a modern integrated [MySQL debugger](https://www.devart.com/dbforge/mysql/studio/code-debugger.html) which allows you to see values of each variable while executing the code.

---
layout: post
title: "How To: Disable All Foreign Keys in Oracle Schema"
description: "TPL/SQL script that enables or disables all foreign key constraints in an Oracle schema, useful for data maintenance, migrations, and synchronization tasks."
tags:
  - command line
  - SQL Server
---

When you perform data maintenance operations, sometimes, it's necessary to disable or enable all foreign keys in the user schema.

Here is the script that solves this task:

```sql
SET serveroutput ON;
/
DECLARE
   /*The name of the schema that should be synchronized.*/
   Schema_Name VARCHAR2(4000) :='type target schema name here';
   /*The operation type:*/
   /*  ON — enable foreign keys;*/
   /*  OFF — disable foreign keys.*/
   ON_OFF VARCHAR2(4000) :='ON';
PROCEDURE CONSTRAINTS_ON_OFF
(Target_Schema_Name IN VARCHAR2, Action IN VARCHAR2:='')
IS
   sql_str VARCHAR2(4000);
   FK_name VARCHAR2(4000);
   var_action VARCHAR2(4000);
CURSOR cCur1 IS
   /*Creating the list of foreign keys that should be disabled/enabled,*/
   /*with creating a command at the same time.*/
   SELECT
      'ALTER TABLE '||OWNER||'.'||
      TABLE_NAME||' '||var_action||' CONSTRAINT '||CONSTRAINT_NAME AS sql_string,
      CONSTRAINT_NAME
   FROM
      ALL_CONSTRAINTS
   WHERE
      CONSTRAINT_TYPE='R' AND OWNER=Target_Schema_Name;
BEGIN
   IF upper(Action)='ON' THEN
       var_action :='ENABLE';
   ELSE
       var_action :='DISABLE';
   END IF;
OPEN cCur1;
   LOOP
      FETCH cCur1 INTO SQL_str,fk_name;
      EXIT WHEN cCur1%NOTFOUND;
      /*Disabling/Enabling foreign keys.*/
      EXECUTE IMMEDIATE SQL_str;
      DBMS_Output.PUT_LINE('Foreign key '||FK_name||' is '||var_action||'d');
   END LOOP;
EXCEPTION
WHEN OTHERS THEN
    BEGIN
        DBMS_Output.PUT_LINE(SQLERRM);
    END;
    CLOSE cCur1;
END;
BEGIN
    CONSTRAINTS_ON_OFF(Schema_Name,ON_OFF);
    /*specify additional calls if necessary*/
END;
/
COMMIT;
/
```

Executing the script with the ON\_OFF parameter set to 'OFF' will lead to disabling foreign keys, and setting it to 'ON' will lead to enabling them.

But you should keep in mind that if data after synchronization contradicts the logic of data integrity on the server side the procedure of disabling and enabling foreign keys will fail.

You can also use this script in express edition of [dbForge Data Compare for Oracle](https://www.devart.com/dbforge/oracle/datacompare/) when transferring master-detail data.

---
layout: post
title: "Loading Oracle LOB Data – Code Samples"
description: "Learn how to load Oracle LOB data (BLOB and CLOB) using PL/SQL code samples, including DBMS_LOB.WRITEAPPEND and file-based methods for handling large data."
tags:
  - data compare
  - Oracle
  - synchronize database
---

Recently we were working over the release of the version 2.0 of [dbForge Data Compare for Oracle](https://www.devart.com/dbforge/oracle/datacompare/). One of the main features added to the new version was synchronization of the LOB data. When we developed the first release of the product we postponed this feature because of challenges of synchronizing LOB data. These challenges caused by nature of data stored in BLOB or CLOB columns. It's large and simply does not fit into SQL script. But we've found the solution.

In this article we will examine basic methods of loading LOB data into a table.

### Using DBMS\_LOB.WRITEAPPEND

The simplest and most convenient method of inserting/updating LOB data in a table is usage of DBMS\_LOB package. In this case, the data gets into a table through additional variables inserted into PL/SQL block.

```sql
DECLARE
  TMP_BLOB BLOB := NULL;
  SRC_CHUNK RAW(12001);
```

TMP\_BLOB variable serves like a buffer storage for a LOB value. A data type of a temporary variable is selected according to a data type of a LOB column, e.g., to load data into a column with CLOB data type, the temporary variable will be of CLOB data type. XMLTYPE is a modification of CLOB data type; therefore its temporary variable will be CLOB. Data is written to buffer in parts with the help of SRC\_CHUNK variable. The variable uses RAW data type, which size is determined by a length of the loaded data part. To create a buffer variable, the following query sequence is used:

```sql
DBMS_LOB.CREATETEMPORARY(TMP_BLOB, TRUE);
DBMS_LOB.OPEN(TMP_BLOB, DBMS_LOB.LOB_READWRITE);
```

After this you can write data into this variable. Here SRC\_CHUNK variable is used, i.e., firstly, data is assigned to the variable, and then, with the help of DBMS\_LOB.WRITEAPPEND function, the data get into TMP\_BLOB variable.

```sql
SRC_CHUNK := 'BBBBBBBBBBBBBBBBBBBBBBBB';
DBMS_LOB.WRITEAPPEND(TMP_BLOB, LENGTH(SRC_CHUNK), SRC_CHUNK);
```

When data is fully loaded into TMP\_BLOB, this variable is used in INSERT/UPDATE clause.

```sql
INSERT INTO TABLE_BLOB(ID, BLB) VALUES (1001, TMP_BLOB);
```

The final script will be:

```sql
DECLARE
  TMP_BLOB BLOB := NULL;
  SRC_CHUNK RAW(12001);
BEGIN
  DBMS_LOB.CREATETEMPORARY(TMP_BLOB, TRUE);
  DBMS_LOB.OPEN(TMP_BLOB, DBMS_LOB.LOB_READWRITE);

 SRC_CHUNK := 'BBBBBBBBBBBBBBBBBBBBBBBB';

  DBMS_LOB.WRITEAPPEND(TMP_BLOB, LENGTH(SRC_CHUNK), SRC_CHUNK);
  DBMS_LOB.CLOSE(TMP_BLOB);

  INSERT INTO TABLE_BLOB(ID, BLB) VALUES (1001, TMP_BLOB);
END;
```

### Loading data through files

This method was designed for cases when DBMS\_LOB.WRITEAPPEND can not be used. It happens when, for example, one query for synchronization exceeds the acceptable size of dedicated storage for the program. In this case a user can load LOB data into a table via files. To do this, the user should have the access to a directory on a server. This directory will be used to create files for synchronization. To load the files, it is required to bing the directory to an Oracle database – this will allow calling the directory from a synchronization script. Calling the directory is done through alias which is assigned by executing the following script:

```sql
CREATE DIRECTORY MY_DIR as 'C:temp'
```

To load data into a table, an auxiliary variable is also used:

```sql
DECLARE
  TMP_BLOB BLOB := NULL;
```

Also an auxiliary function LOAD\_BLOB\_FROM\_FILE is declared:

```sql
FUNCTION LOAD_BLOB_FROM_FILE(FILENAME VARCHAR2, DIRECTORYNAME VARCHAR2)
  RETURN BLOB
IS
  FILECONTENT BLOB := NULL;
  SRC_FILE BFILE := BFILENAME(DIRECTORYNAME,FILENAME);
  OFFSET NUMBER := 1;
BEGIN
  DBMS_LOB.CREATETEMPORARY(FILECONTENT,TRUE,DBMS_LOB.SESSION);
  DBMS_LOB.FILEOPEN(SRC_FILE,DBMS_LOB.FILE_READONLY);
  DBMS_LOB.LOADBLOBFROMFILE (FILECONTENT, SRC_FILE, DBMS_LOB.GETLENGTH(SRC_FILE),
    OFFSET, OFFSET);
  DBMS_LOB.FILECLOSE(SRC_FILE);
  RETURN FILECONTENT;
END;
```

This function loads data from a file into BLOB variable. The main function is DBMS\_LOB.LOADBLOBFROMFILE that reads the file's content. DBMS\_LOB.LOADСLOBFROMFILE function is used for loading text data. Its input parameter is a file encoding name. Calling LOAD\_BLOB\_FROM\_FILE returns a LOB value that should be sent to a database.

```sql
  TMP_BLOB := LOAD_BLOB_FROM_FILE('BLB.bin', 'MY_DIR');
  INSERT INTO TABLE_BLOB(ID, BLB) VALUES (2000, TMP_BLOB);
```

The final script will be:

```sql
DECLARE
  TMP_BLOB BLOB := NULL;

  FUNCTION LOAD_BLOB_FROM_FILE(FILENAME VARCHAR2, DIRECTORYNAME VARCHAR2)
    RETURN BLOB
  IS
    FILECONTENT BLOB := NULL;
    SRC_FILE BFILE := BFILENAME(DIRECTORYNAME,FILENAME);
    OFFSET NUMBER := 1;
  BEGIN
    DBMS_LOB.CREATETEMPORARY(FILECONTENT,TRUE,DBMS_LOB.SESSION);
    DBMS_LOB.FILEOPEN(SRC_FILE,DBMS_LOB.FILE_READONLY);
    DBMS_LOB.LOADBLOBFROMFILE (FILECONTENT, SRC_FILE, DBMS_LOB.GETLENGTH(SRC_FILE),
      OFFSET, OFFSET);
    DBMS_LOB.FILECLOSE(SRC_FILE);
    RETURN FILECONTENT;
  END;

BEGIN
  TMP_BLOB := LOAD_BLOB_FROM_FILE('BLB.bin', 'DC1_DIR');
  INSERT INTO TABLE_BLOB_LARGE(ID, BLB) VALUES(2000, TMP_BLOB);
END;
```

### Problems in CLOB and NCLOB data synchronization on Oracle 8

Challenges happen as earlier Oracle versions do not have LOADСLOBFROMFILE function of DBMS\_LOB package. This forces the usage of DBMS\_LOB.LOADFROMFILE function that does not have parameters to manage text encoding. As a result, data can be inserted incorrectly if any non-Latin symbols exist.

### Conclusion

The illustrated methods have their pros and cons. Usage of DBMS\_LOB.WRITEAPPEND allows loading data via only a script, howerver, in case of dealing with large bulks of data, it will be impossible to execute such a scipt. Loading LOB from a file also has a defect as moving a file to a server pc brings much inconvenience.

---
layout: post
title: "Adding Timestamp to a Filename"
description: "This article describes how to add time and date to a filename using the command line."
tags:
  - command line
  - SQL Server
---

**Summary**: This article describes how to add time and date to a filename using the command line.

Sometimes it is crucial to append time and date to the name of a file. For example, we would like to have separate log files for each execution of data synchronization script. To prevent file overriding, we want to specify date and time in the name of each log file.

Generally, the solution is quite simple — we should use the %date% and %time% built-in variables, that display date and time based on the regional coding. In our case, the variables returned the following values:

- for date

```bat
echo %date%
Thu 01/22/2015
```

- for time

```bat
echo %time%
16:20:08.80
```

In most cases, we do not need to put the entire date and time, as it is specified above. In our case, we wish to output date in the MM/DD/YY format, and time in the HH:MM:SS format. To achieve this, we will create the .bat file, and modify system variables with help of the SET command:

```bat
echo %date%
SET mm=%date:~4,2%
SET dd=%date:~7,2%
SET yy=%date:~12,2%
echo %time%
SET hh=%time:~0,2%
SET min=%time:~3,2%
SET ss=%time:~6,2%
```

**Note**: digits after tilde specify the position and the number of characters we want to use, for example %date:~4,2% mean that we want to output two characters, starting after the fourth character (numbering begins from 0).

Here what command line returned after execution of our .bat file:

```bat
SET mm=01
SET dd=22
SET yy=15
SET hh=18
SET min=09
SET ss=12
```

Now, all we have to do is to put it all together to the required script after the /log argument:

```bat
/log:"D:\dbForge\logfile\_%mm%%dd%%yy%\_%hh%%min%%ss%.log"
```

The execution of the .bat file should return the following:

```bat
logfile_012215_181259.log
```

There is also an alternative way to perform the task without the .bat file and the SET command. In this case, you should modify system variables right in the script. For example:

```bat
/log:"D:\dbForge\logfile%date:~4,2%%date:~7,2%%date:~12,2%\_%time:~0,2%%time:~3,2%%time:~6,2%.log"
```

To sum everything up, let's use the timestamp code in practice. Let's compare and synchronize data through the command line by means of [dbForge Data Compare for SQL Server](https://www.devart.com/dbforge/sql/datacompare/), and add timestamp to the logfile name:

![Commandline timestamp](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/command-line-timestamp.png)

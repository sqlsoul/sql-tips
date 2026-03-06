---
layout: post
title: "Find and Delete Incomplete Open Transactions in SQL Server"
description: "Learn how to find and delete incomplete open transactions in SQL Server databases more quickly and efficiently with dbForge Studio for SQL Server's functionality."
tags:
  - sql complete
  - SQL Server
  - sql server transactions
---

By incomplete transaction, we shall basically understand an active (running) transaction that doesn't have any active (running) queries for some long period of time T.

The general algorithm for deleting incomplete transactions:

1. Create two tables: one table to store and analyze information about current incomplete transactions and the second one to archive the transactions selected from the first table according to the delete actions for subsequent analysis.

2. Gather information about transactions and their sessions that have no queries, i.e., transactions launched and left forgotten within a certain timespan T.

3. Update the table containing current incomplete transactions from step 1 (if an incomplete transaction acquires an active request, then such a transaction is no longer considered to be incomplete and is deleted from the table).

4. Identify the sessions to be killed (a session has at least one incomplete transaction mentioned in the table from step 1 and there are no queries running for that session as well).

5. Archive the information you are going to delete (details about the sessions, connections, and transactions that are to be killed).

6. Kill the selected sessions.

7. Delete the processed entries along with those that cannot be deleted and have been in the table from step 1 for too long.

We also presented the process of [creating a CRUD stored procedure](https://blog.devart.com/find-and-delete-sql-server-incomplete-open-transactions.html).

Let's now review a number of settings that help you work more efficiently.

1. [Tabs Coloring functionality](#Tabs_Coloring)
2. [Import and Export Reset Settings](#Reset_Settings)
3. [Restoring Documents Sessions](#Restoring_Sessions)

## **Marking Execution Environment using colors**

Let's create two windows: one for the testing stand and one for the production environment. Now let's paint every window in proper colors (green for the testing environment, red for the production environment). To do this, we need to right-click on the proper window, select Tabs Color in a drop-down menu, and select a color.

![Using Color Marking for Tabs](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/1find-and-delete-incomplete-open-transactions-in-ms-sql-server.png)

*Pic. 1. Changing the document window colors*

Now we get the colored tabs depending on colors that we selected:

![Results of Tab Coloring](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/2find-and-delete-incomplete-open-transactions-in-ms-sql-server.png)

*Pic. 2. The result of changing the script windows colors of the documents*

This functionality can also be changed in SQL Complete\Options\Tabs Color:

![Tabs Color Sub-menu in Options](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/3find-and-delete-incomplete-open-transactions-in-ms-sql-server.png)

*Pic.3. Tabs and instances of color settings*

Here you can change, add, or delete colors for specific hosts.

[Tabs Coloring](https://www.devart.com/dbforge/sql/sqlcomplete/productivity-extension.html#tabs_coloring) functionality not only allows us to discern environment types (testing, production, etc.) but does the same job for important MS SQL instances.

## **Importing, exporting, and reset to default settings in SQL Complete**

Among other settings, SQL Complete lets you adjust import and export parameters:

![Navigating to Import and Export Parameters](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/4find-and-delete-incomplete-open-transactions-in-ms-sql-server.png)

*Pic.4. SQL Complete settings import and export*

After you click on "Import and Export Settings…", the following window pops up:

![Import and Export Settings Wizard](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/5find-and-delete-incomplete-open-transactions-in-ms-sql-server.png)

Pic.5. SQL Complete settings export

This window allows for selecting either import, or export, or reset to default settings. In our case, we go for settings export and press the "Next" button. Then we check proper sections and press the "Next" button:

![Selecting Data to export](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/6find-and-delete-incomplete-open-transactions-in-ms-sql-server.png)

*Pic.6. Selecting the exported data*

Then we select the "save" folder, make the file's name, and press the "Execute" button:

![Setting the Export File Path](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/7find-and-delete-incomplete-open-transactions-in-ms-sql-server.png)

*Pic.7. Setting up the file's destination for exported settings and launching the export process*

Once the export is done, we receive a message saying that it's over and press the "Finish" button.

![Finishing the Export Process](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/8find-and-delete-incomplete-open-transactions-in-ms-sql-server.png)

*Pic.8. Finishing the settings export process*

The import process goes the same way.

Let's now return to our new stored procedure in two windows created above.

You can read more info about importing, exporting, and resetting to default your SQL Complete settings [here](https://docs.devart.com/sqlcomplete/setting-up-sql-complete/importing-and-exporting-settings.html).

## **Restoring documents**

To restore your sessions based on windows with scripts, you should use the **SQL Complete\Document Sessions** command:

![Navigating to Documents Sessions Window](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/9find-and-delete-incomplete-open-transactions-in-ms-sql-server.png)

*Pic.9. Selecting the "Documents Sessions" command in the SQL Complete Main Menu*

Once you selected it, the restorable sessions window pops up.

![Selecting the Sessions to Restore](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/10find-and-delete-incomplete-open-transactions-in-ms-sql-server.png)

*Pic.10. Selecting the sessions to restore*

Right-click on the necessary document to restore it:

![Restoring the Selected Document](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/11find-and-delete-incomplete-open-transactions-in-ms-sql-server.png)

*Pic.11. Restoring the selected document*

You can find more information about the Documents Sessions functionality [here](https://docs.devart.com/sqlcomplete/tab-and-document-management/restoring-documents.html).

If we accidentally close a window with a script, this won't be a problem. By using the **SQL Complete\Restore Last Closed Document** command, you can re-open that window and not lose any of your important scripts:

![Restoring Last Closed Document](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/12find-and-delete-incomplete-open-transactions-in-ms-sql-server.png)

*Pic.12. Selecting the "Restore Last Closed Document" command in the SQL Complete Main Menu*

By using the **SQL Complete\Recently Closed Documents** command, we can do the same trick of restoring any recently closed file:

![Restoring Recently Closed Documents](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/13find-and-delete-incomplete-open-transactions-in-ms-sql-server.png)

*Pic.13. Selecting the "Recently Closed Documents" command in the SQL Complete Main Menu*

In this article, we reviewed a number of [productivity features](https://www.devart.com/dbforge/sql/sqlcomplete/productivity-extension.html) that help you work more efficiently in the process of implementing algorithms for deleting incomplete transactions in SQL Server with the help of SQL Complete.

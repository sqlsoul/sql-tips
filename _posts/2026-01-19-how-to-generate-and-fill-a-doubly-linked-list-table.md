---
layout: post
title: "How To Generate and Fill a Doubly Linked List Table"
description: "In this article, we’ll learn how to fill a doubly linked list table with SQL test data. In such tables, there can be various independent chains unified by unique tags into groups."
tags:
  - data generator
  - SQL Server
---

### Introduction

In this article, we'll learn how to fill a doubly linked list table with SQL test data. In such tables, there can be various independent chains unified by unique tags into groups.

You can see an example of a doubly linked list table below:

![Doubly Linked List Example](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/1generate-and-fill-a-doubly-linked-list-table.png)

Of course, to fill such a table with data, you could write a rather simple SQL script utilizing the WHILE statement. However, this method has some disadvantages. First of all, the data created in this way will look very unrealistic. In addition, a doubly linked list table may be connected with other tables (i.e., it may have foreign keys), which makes filling it with data much more difficult.

This is why we'll use [dbForge Data Generator for SQL Server](https://www.devart.com/dbforge/sql/data-generator/) to generate data for our table. For the sake of this example, we'll create one doubly linked list table with no foreign keys.

### The process

Here are the steps we will need to take:

1. Run [dbForge Data Generator](https://www.devart.com/dbforge/sql/data-generator/download.html).

2. Create a new database called TestDB and, inside of it, a new table.

```sql
CREATE TABLE linkedListTable(
	id INT NULL,
	fullName NVARCHAR (100) NULL,
  email NVARCHAR(50) NULL,
	previous_id int NULL,
	next_id INT NULL,
  chainGroup INT
);
```

![Table Creation Script](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/2generate-and-fill-a-doubly-linked-list-table.png)

In this table, the **chainGroup** field specifies the chain group value, **previous\_id** points to the previous element in the chain, and **next\_id** points to the chain's next element.

3. Click "New Data Generation" on the database. In the Data Generation Wizard, you can leave all options at their default values.

4. For the **previous\_id**, **next\_id** and **chainGroup** columns, we will need to uncheck the **Include NULL** option.

![Include NULL Option](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/3generate-and-fill-a-doubly-linked-list-table.png)

5. Next, select the Python generator for these columns:

![Python Generator Option](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/4generate-and-fill-a-doubly-linked-list-table.png)

6. For the **previous\_id** column, we'll need to enter the following script in the Python editor:

```python
import clr
clr.AddReference("System")
from System import DBNull
from System import Random

random = Random(1)

def main():
  i = 0

  max_possible_count_in_chain = 8 # change this value if you need
  
  while True:
     items_count_in_chain = random.Next(1, max_possible_count_in_chain)
       
     while True:
       if (items_count_in_chain == 0 or i == 0):
	      yield DBNull.Value
		 
       if (items_count_in_chain == 0):
	     i+= 1
	     break
       i+= 1
       yield i
       items_count_in_chain -= 1

main()
```

This is the result we'll get:

![Generation Options Result](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/5generate-and-fill-a-doubly-linked-list-table.png)

7. Next, for the **next\_id** column, enter the following script in the Python editor:

```python
import clr
clr.AddReference("System")
from System import DBNull
from System import Random

random = Random(1)

def main():
  i = 1
  max_possible_count_in_chain = 8 # change this value if you need

  while True:
     items_count_in_chain = random.Next(1, max_possible_count_in_chain)
       
     while True:
       if (items_count_in_chain == 0):
	      yield DBNull.Value
		 
       if (items_count_in_chain == 0):
	     i+= 1
	     break
       i+= 1
       yield i
       items_count_in_chain -= 1

main()
```

8. For the **chainGroup** column, use the following script:

```python
import clr
clr.AddReference("System")
from System import DBNull
from System import Random

random = Random(1)

def main():
  i = 0
  max_possible_count_in_chain = 8 # change this value if you need
    
  while True:
     i +=1 
     items_count_in_chain = random.Next(1, max_possible_count_in_chain) + 1
	 
     while True:
        if (items_count_in_chain == 0):
           break
        yield i
        items_count_in_chain -= 1
	   
main()
```

9. As a result, we'll get a Data Preview like the one seen on the following screenshot. That's it!

![Resulting Table](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/6generate-and-fill-a-doubly-linked-list-table.png)

10. For this particular example, the largest possible chain length was set to 8. However, you can set this value according to your needs. To do this, you would need to make changes in all three columns – **previous\_id**, **next\_id** and **chainGroup**.

![Max Chain Length Setup](https://raw.githubusercontent.com/sqlsoul/sql-tips/refs/heads/main/assets/media/7generate-and-fill-a-doubly-linked-list-table.png)

### Summary

In this article, we looked at a way of filling a doubly linked list table with randomly generated SQL test data. By configuring the Python generator, you can generate data for tables based on other database structures.

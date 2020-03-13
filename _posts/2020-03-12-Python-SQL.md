---
title: 'Connecting MySql database using MySQLdb'
date: 2020-03-12
permalink: /posts/2020/03/post-SQL-py/
tags:
  - SQL
  - python
  - database
---

Database can be accessed using Python database API (DB-API). Therefore, you can make use of the very rich Python ecosystem for data analysis.

## Terminologies

  - DB-API: The communicator between Python and DBMS
  - Connection objects: database connections; manage transactions
  - Cursor objects: database querries

## basic syntax

<pre>
from dbmodule import connection
# create connection object
connection =connect('host','user_name','pswd','databasename');
# create a cursor object
cursor =connection.cursor();
# run querries
cursor.execute('select * from mytable');
results = cursor.fetchall();
# free resources
cursor.close();
connection.close();
</pre>

## Using MySQLdb
First of all, install the `MySQL-python` driver if it does not exist. 
<pre>
# using conda
$ conda install MySQL-python
# using pip
pip install MySQL-python [--user]
</pre>

The template as below:
<pre>
import MySQLdb

db = MySQLdb.connect(host='your host name',  # your host name is often 'localhost'
                 user='your username',            
                 passwd='your password',  
                 db='your database')
cur = db.cursor()

# to apply SQL
q= 'select * from table'
cur.execute(q)

for row in cur.fetchall():
print row

db.close()
</pre>



















---
layout: post
title: "SQL Injection Tutorial"
date: 2016-04-15
---

SQL stands for **S**tructrued **Q**uery **L**anguage.

Some properties of SQL are:

1. Tabular Database
2. Organized Layout (schema)

There are many different flavors of SQL, such as:

1. PostreSQL
2. MSSQL
3. MySQL
4. MariaDB
5. SQLite
6. ... and many, many more

Lots and lots of large products use SQL:

1. Android
2. iOS
3. Mozilla Firefox
4. Oracle
5. Microsoft
6. A lot... of websites...

As a demonstration, we can use Python's SQLite3 library as a quick demonstration:

```
>>> import sqlite3
>>> conn = sqlite3.connect("mydb.db")
>>> c = conn.cursor()
>>> c.execute("CREATE TABLE table1(a text, b integer)")
<sqlite3.Cursor object at 0x10aad31f0>
>>> c.execute("INSERT INTO table1 VALUES ('yo', 1)")
<sqlite3.Cursor object at 0x10aad31f0>
>>> c.execute("SELECT * FROM table1").fetchall()
[(u'yo', 1)]
```

The above creates a database called `mydb.db`. Then it creates a `cursor` object
that will "type" out commands to be executed in a `SQLite3`. This is just the
way Python implemented the SQLite library. The string we input into `c.execute`
is an SQLite command.

```
CREATE TABLE table1(a text, b integer)
```

This creates a table in the database called `table1` with the fields `a`, an
integer, and `b`, a text field.

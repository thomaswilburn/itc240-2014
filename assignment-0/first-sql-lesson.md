SQL Lesson #1
=============

A web app that can only present hard-coded, hand-written data isn't very interesting. Really interesting web apps offer persistence: they can store information, then retrieve it at a later time. Although it's possible for PHP scripts to store data in flat files, in RAM, or in user cookies, it's most common to use a database of some kind. For this class, we'll be using MySQL, but there are plenty of other options available, such as PostgreSQL or SQLite.

What is SQL anyway? It's a "structured query language," but it's not like the other programming languages that we tend to use. In procedural languages, like PHP, we tell the computer exactly what we want it to do, in the order that we want it done. In SQL, by contrast, we describe the result we want, and the database software figures out a _query plan_ to get the desired data from the underlying table.

For example, let's say that we have a set of data represented by this table:

| class | room | instructor |
--------|------|-------------
| web150 | BE3156 | Wilburn |
| itc260 | BE3175 | Newman |
| mic102 | BE3156 | Staff |

If we wanted to only get the classes that took place in BE3156 using PHP, we'd have to look through all the results, like so:

```php
$result = [];
foreach ($classes as $class) {
  if ($class["room"] == "BE3156") {
    $result[] = $class;
  }
}
```

But in SQL, it's a lot easier to express what we want:

```
SELECT * FROM classes WHERE room = 'BE3156';
```

So elegant! So simple! When it comes to retrieving and updating organized data, it doesn't get much better than SQL. There are many flavors of SQL, depending on the database that you're using, but they all work roughly the same way, and most of the time skills will transfer from one to the other.

Let's set up a simple database table that we can use for some queries from a web page. First, go [set up your MySQL account on Zephir](https://zephir.seattlecentral.edu/mysql). Then log in with PuTTY and run the following command to log into your personal database:

```
mysql -D [username] -p
```

The result should be a `mysql>` prompt where you can type. The `mysql` program on Zephir is a command line for accessing databases--a REPL (read-execute-print loop) just like the PHP interpreter mode we used in class to test simple ideas. If it seems like we're using the command line a lot, that's because many of our most technical tools for web development run on the command line for its flexibility and precision. It's in your best interest to become comfortable with it, hence our emphasis in this class.

First things first, we need to make a table. We're going to start with a simple set of names combined with numbers. First, we need to set up the columns for our table, so we're going to run a `CREATE TABLE` command. This command includes the names and types of each column in the parentheses: one that's a text column for the names, and another that's integers (whole numbers) for their ID numbers.

```
mysql> CREATE TABLE test (name TEXT, id INTEGER);
Query OK, 0 rows affected (0.01 sec)
```

As a matter of routine, I usually capitalize SQL commands like `CREATE TABLE` or `TEXT` to distinguish them from table or column names, but the language itself is case-insensitive. To see the table we just created, let's type two commands, `SHOW TABLES` and `SHOW COLUMNS IN test`.

```
mysql> show tables;
+--------------------+
| Tables_in_twilburn |
+--------------------+
| test               | 
+--------------------+
1 row in set (0.00 sec)

mysql> show columns in test;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| name  | text    | YES  |     | NULL    |       | 
| id    | int(11) | YES  |     | NULL    |       | 
+-------+---------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```

Now we have a table, but there's nothing in it. We need to put some data into this table. In SQL, we insert data by, simply enough, writing an `INSERT` statement. When we insert data, we have to let it know which table we want to insert rows into, then write out the rows in parentheses, with text in single quotes only. So to insert four rows with new names and ID numbers, we can write:

```
INSERT INTO test VALUES ('Alice', 1),('Bob', 2),('Charlie', 3),('Dana', 4);
```

You should see a line saying something like "Query OK, 4 rows affected" to confirm that the `INSERT` was successful. Now let's take a look at the data. We'll use a `SELECT` statement to see what we put into the database. I wrote a `SELECT` statement above, when I talked about getting certain information from a table, but in this one we're just going to ask for everything:

```
msyql> SELECT * FROM test;
+---------+------+
| name    | id   |
+---------+------+
| Alice   |    1 | 
| Bob     |    2 | 
| Charlie |    3 | 
| Dana    |    4 | 
+---------+------+
4 rows in set (0.00 sec)
```

The `*` in the statement means that we want all the columns in `test`. If we want, we can ask only for certain columns by specifying their names. The following statement will only give us the `name` column, but not `id`.

```
SELECT name FROM test;
```

By default, a `SELECT` statement will give us back all the rows that match our condition. Since we didn't specify any conditions in this statement, other than which columns we wanted, it gives us all of the rows in the table. But we can add a `WHERE` clause to specify which rows we want and filter it down a little. 

```
mysql> SELECT * FROM test WHERE id > 2;
+---------+------+
| name    | id   |
+---------+------+
| Charlie |    3 | 
| Dana    |    4 | 
+---------+------+

mysql> SELECT * FROM test WHERE name = 'Alice';
+-------+------+
| name  | id   |
+-------+------+
| Alice |    1 | 
+-------+------+
```

You'll notice that we can test in most of the same ways that conditionals can check values in PHP, such as `>`, `<`, `<=`, and `>=`. When testing equality in SQL, however, we don't have to write `==`, because the language is built in such a way that it always knows the difference between assigning values and testing them. You should use `=` for "is equal to" instead.

`SELECT` statements can get very complex, combining multiple tables into a single result or filtering in interesting ways, but most of the statements we'll write will be very similar to this. You can look at the documentation for MySQL `SELECT` statements [here](http://dev.mysql.com/doc/refman/5.0/en/select.html), but it's pretty complex, so let's break down a few other simple operations.

Let's look at our table again, but this time let's sort the rows. You can use an `ORDER BY` clause to sort your results by any given column, with `ASC` or `DESC` to pick between ascending or descending order, respectively:

```
SELECT * FROM test WHERE id > 1 ORDER BY name DESC;
+---------+------+
| name    | id   |
+---------+------+
| Dana    |    4 | 
| Charlie |    3 | 
| Bob     |    2 | 
+---------+------+
3 rows in set (0.00 sec)
```

As you can see, we can also combine the `ORDER BY` clause with our `WHERE` clause, although the `ORDER BY` must come last.

The flow chart for `SELECT` statements, including all possible permutations, is a rat's nest of interconnected lines. But most `SELECT` statements can be expressed in much simpler terms. The following table lays out the parts of a `SELECT` in order, with optional clauses marked.

Clause           | Optional? | Description
-----------------|-----------|------------
SELECT [columns] | No        | Always start with the SELECT keyword, followed by a comma-separated list of columns (or the `*` character if you want all the columns in the table.
FROM [table]     | No        | We may have many tables in a database, so we need to state explicitly which one we're going to pull data from
WHERE [condition]| Yes       | If you want to filter data, only rows that match the WHERE clause will be returned from the database
ORDER BY [column] ASC/DESC | Yes | You can sort data by using an ORDER BY clause, specifying the column you want to sort by, and the direction (ascending or descending)
LIMIT [number]   | Yes       | It's possible to only return a few rows by using LIMIT and a number. Combined with ORDER BY, you can get only the top or bottom few rows in a database this way.
OFFSET [number]  | Yes       | OFFSET allows you to move the starting point of a LIMIT. See below for examples.

Here are a few sample SQL queries from a hypothetical RSS feed reader, and an explanation of what they mean:

Query | Translation
------|------------
`SELECT * FROM stories;` | Get all items and columns from the "stories" table.
`SELECT id, title FROM stories;` | Get two columns of data from a table named "stories"
`SELECT id, title FROM stories WHERE feed = 137;` | Get two columns from "stories" where the feed ID matches the value `137`.
`SELECT title, content FROM stories WHERE read = false LIMIT 10` | Get 10 unread stories (title and content) from the database. These will be random stories, since we didn't feature a sort order
`SELECT title, content FROM stories WHERE read = false ORDER BY published_date DESC LIMIT 10;` | Get the last 10 unread stories in the database (ordered by publication date in descending order with a limit of 10).

SQL is an entirely new programming language, in addition to HTML, CSS, and PHP. Working on the web stack requires us to be able to move between these languages easily, so it's not for the faint of heart. Take some time and practice inserting and selecting rows from your database, or creating new tables. Later, we'll learn how to change rows, delete them, and even delete entire tables if necessary.

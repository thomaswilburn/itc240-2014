Basic SQL Security
==================

It's my goal in this class to make sure that we're learning good security practices as we go, since all good application design should include security as a first principle. After all, your users (and your stockholders, at most companies) are counting on you. There are two major points of weakness in the security of most web applications: cross-site scripting, and SQL injection. We learned about the former in class, and talked about mitigating it by using the `htmlentities()` function on any output that comes directly from the user. Now we'll learn about the latter.

Imagine that you're building SQL statements as strings in PHP before you send them to the database. You might have a few lines of code for performing user lookup:

```php
$input = $_REQUEST["username"];
$query = "SELECT * FROM users WHERE username = '$input';"; //use double-quoted strings to put the username into the query.
```

If the user provides a normal value for `username`, all is well, and your SQL (with `$input` substituted into the string) looks like this:

```sql
SELECT * FROM users WHERE username = 'twilburn';
```

But what happens if, instead of providing a valid username, they include the following string: `' OR 1 = 1 --`? Now your query looks more like the following:

```sql
SELECT * FROM users WHERE username = '' OR 1 = 1; --';
```

The `--` sequence in SQL signifies a line comment, similar to `//` in PHP, so now your query will select all users (after all, `1 = 1` is always true in the `WHERE` clause, so all rows will be returned from the database). By including their own closing quote and then writing in some valid SQL code, this user just managed to get access to all your user records. That's probably a bad thing. 

It's especially bad, because they don't just have to write a SELECT statement. You may have seen this XKCD comic floating around:

<img src="http://imgs.xkcd.com/comics/exploits_of_a_mom.png">

As you prepare for Tuesday's class, be thinking about how you would protect yourself from Little Bobby Tables. How can we stop users from being able to inject their own SQL into our queries?

You may find it helpful to read up more on SQL injection. The following pages are great resources on the topic:
* [Wikipedia's page on SQL injection](http://en.wikipedia.org/wiki/SQL_injection) covers the basics, lists some solutions, and also details a number of infamous failures. 
* As always the Open Web Application Security Project [offers an extensive guide](https://www.owasp.org/index.php/SQL_Injection) on this and other attacks, including multiple cheat sheets on preventing attacks.

It may seem paranoid to introduce security concerns while many of you are still grappling with the basics of PHP and SQL. However, remember the following: when you place a page on the Internet, it's not just going to be viewed by your preferred users. Hackers will regularly set up bots to visit random pages online and attempt common attacks like SQL injection. The reality of the modern web is simply that we cannot assume that a page is safe just because it's small, or because we're newbies to coding. The only way to be safe is to do it right from the start.

A quick anecdote, to drive this point home on a day of protests in Seattle: when I worked at the World Bank Institute in 2006, we were hacked by a Turkish group that didn't approve of the appointment of Paul Wolfowitz (a prominent member of the Bush administration and architect of the war in Iraq) to the presidency of the World Bank. Ironically, of course, most Bank staff were equally unhappy about Wolfowitz's appointment, but the hackers didn't ask our opinion. Instead, they took over our server and posted up insulting messages about his socks (I am not making this up).

How did they get in? It turns out that our department's web programmer had left one of the form inputs on the site unprotected, and they were able to orchestrate a SQL injection attack. As soon as they got access, they painstakingly built a list of tables and databases on the server by inserting their own commands, then dumped all of our data out and inserted their own in its place. The entire situation was extremely embarrassing, and cost my team several weeks of productivity as the site was rebuilt and audited for security. 

The World Bank is obviously a high-profile target, but any site can be attacked. Start thinking about security now, and the owners of your sites (whether they are employers, volunteer organizations, family members, or even yourself) will be grateful for the effort.



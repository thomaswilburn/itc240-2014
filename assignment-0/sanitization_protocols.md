The Ins and Outs of Sanitized Input and Output
==============================================

Creating secure web pages is difficult, not because of the theory required or the high level of technical skill, but merely because it asks developers to be paranoid and painstaking about their inputs and outputs. For most of our CRUD applications, however, the flow will be similar to the diagram below, with prepared statements used to regulate inputs into the database, and `htmlentities()` normalizing its output to protect against cross-site scripting.

```
+----------------------+
|                      |
|   Browser (HTML)     | <-------------+
|                      |               |
+----------+-----------+               |
           |                           +
      Form inputs               htmlentities();
           |                           ^
           |                           |
           v                           |
  Prepare/Bind/Execute                 |
           +                           |
           |                       Output from
           |                         SELECT
+----------+-----------+               |
|                      |               |
|    MySQL Database    +---------------+
|                      |
+----------------------+
```

It may be helpful to think about security as a kind of rock-paper-scissors. After all, each stage of the application is vulnerable to a different kind of attack, and we use different tools to protect each one:

| Layer | Input from | Vulnerable to | Protected by |
+-------+------------+---------------+--------------+
| HTML | User input (query string or posted data), database output | Malicious `<script>` and `<style>` tags | `htmlentities()` |
| SQL | User input from PHP | SQL commands injected into the input | Prepared statements and whitelists |

Let's handle each of these in turn. We'll start by looking at the HTML layer, and protect ourselves against malicious XSS (cross-site scripting) attacks. Then we'll figure out how to keep our database queries safe through prepared queries and whitelists. Finally, we'll figure out how to think about security as a general process, not just specific technologies.

Attacking the Browser
---------------------

When we talk about attacking the browser, we're mostly describing the process of getting the page to run code that the author didn't write. For example, if we can sneak a `<script>` tag into the page, we can run JavaScript that would steal the user's cookies and send them to someone else. Any foothold is enough, since it only takes a single line of code to include another JavaScript file, this one with whatever code we want:

```js
document.body.innerHTML += "<script src='evil.js'></script>";
```

For this reason, we need to prevent users from ever being able to inject their content into the page where it will be treated as HTML/CSS/JavaScript and not as text. To do this, we use the `htmlentities()` function to turn special HTML characters into their escaped versions. The above code, for example will turn into:

```js
document.body.innerHTML += &quot;&lt;script src=&quot;evil.js'&gt;&lt;/script&gt&quot;;
```

Now, instead of actually running this JavaScript, it will add the code to the body as text, including the angle brackets and quote characters--annoying, perhaps, but not actually dangerous.

There are multiple ways that a user can get their content onto the page:

* They can include it in the query string, if parts of it are exposed on the page
* They can send it via a form POST
* If they can get it into the database, it won't be directly included, but may be loaded from the database at a later time and shown
* They can tamper with their cookies via the browser console
* If part of your application saves files to the server hard drive, they may be able to change those files and then link to them.

In all of these cases, the values should be escaped by filtering it through the `htmlentities()` function before sending it to the page. Do not assume that just because you got the value from the server or the database, that it's clean. Treat all values as possibly hostile unless they're written _explicitly_ into the code as a number, a boolean, or a single-quoted string.

Attacking the Database
----------------------

Your database is actually a two-fold infection vector. It can be used to attack the browser, by inserting bad HTML into it--the database doesn't actually care about or run HTML, so this is safe for the server but potentially hazardous for the client, as detailed above. However, we can also be vulnerable to attacks on the database itself, by injecting SQL.

Injected SQL itself is not difficult to understand. Since SQL commands are just text that is run as a kind of scripting language, they're vulnerable to interference by altering those commands with new text values. These attacks are often small and hard to execute--an attacker may be able to get a single password at a time, or determine the names of some of your database tables--but they are easy to automate, and can lead to larger vulnerabilities.

To prevent SQL attacks, we forewarn the database about the incoming query by "preparing" it. Effectively, we're providing the basic form of the query, with some blanks left inside, and then filling in the blanks with user values. If the inserted values cause the query to change from its prepared form, the database rejects the query. It will also take care to sanitize values based on their type.

Here's a sample command being prepared, bound and executed:

```php
$prepared_query = $mysql->prepare('SELECT * FROM table WHERE name = ?;');
$prepared_query->bind_param("s", $name);
$prepared_query->execute();
```

In the first line, we tell the database about the query that we intend to run, leaving `?` placeholders where we'll fill in our untrustworthy user values. Then, on the second line, we provide those values separately, including a map of their "type" (usually "s" for "string", "i" for "integer", or "d" for floating point "double-precision" numbers). Finally, we call the `execute` function on the query to run it with the bound values.

Prepared statements can protect the database from user input that may contain malicious SQL commands, but they can be problematic when it comes to dynamic queries, such as those using `ORDER BY` for a custom sort. In this case, we should use a "whitelist" to make sure that only the expected values can be passed in, and then substitute those values into the query before preparation using double quotes:

```php
$sort = $_REQUEST["sort_by"]; //possibly-dangerous user input

//here's our list of "acceptable" sort values
$sortable = [
  'name' => 1, //the value doesn't actually matter, only the key
  'address' => 1,
  'student_id' => 1
];

//check to see if the sort provided is OK
if (!isset($sortable[$sort])) {
  //if it's not in the whitelist, use a default instead
  $sort = 'name';
}

//now put that value in using double quotes to substitute
$query = "SELECT * FROM students ORDER BY $sort;";
```

Whitelists can be useful in both SQL and PHP, but I recommend only using them for the case of `ORDER BY` clauses. Elsewhere, they are often too limiting, and too easy to screw up.

Thinking Securely
-----------------

At this point, you should have the tools to protect your HTML against malicious tags, and your database against malicious code. But how do we learn to think about security safely? To do so, we must be paranoid about our data, refusing to trust any of it, and understanding the concept of "tainted" data flows.

A good web application is extremely paranoid about the data it takes in and what it presents. Do not assume that the user inputs are safe, obviously, but you should also be distrustful of inputs that you think of as "under your control." For example, we might assume that the data pulled from the server, either via a file or the database, is safe, but this may not be the case if the user managed to get bad input past your filters. It may also not be safe to assume that a result is a number or other type that doesn't need to be converted--since PHP is a "weakly-typed" language, there's nothing stopping a variable that should contain a number from containing a string instead.

In most modern template languages, the basic "echo" tags automatically escape HTML before placing it in the page, and developers must manually choose to perform "unsafe" echo without sanitization. Unfortunately, PHP is not a modern template language, and its basic `<?= $value ?>` tag does not perform any sanitization. When writing raw PHP, therefore, we should _always_ call `htmlentities()` inside these tags, or anywhere else that we echo values to the page.

More importantly, we should think about our program's data as a flow which can be "tainted" at any point where it comes in contact with the user. Take the following code, for example:

```php
$cost = 12;
$quantity = $_REQUEST['quantity'];
$costString = "Books: $cost dollars each";
$total = $cost * $quantity;
$totalString = "Total: $" . $total;
```

In this code, we have data flow from various values and inputs into other values. The value of the `$costString` variable depends on the value of `$cost`, and so we would say that there is a data flow from `$cost` to `$costString`. Likewise, `$totalString` depends on `$total`, which itself depends on `$cost` and `$quantity`. The data flow for this code therefor looks a little like this:

```
12--------------->$cost----+------> $costString
                           |
                           +-----------+
                                       v
$_REQUEST-------->$quantity-------->$total
                                       |
                      +----------------+
                      |
                      v
                $totalString
```

We wrote `12` directly into the code, so the value of `$cost` is considered "clean" (it's completely in our control), and so is `$costString` (because it only depends on clean values). The `$_REQUEST` array, however, is controlled by the user, and must be considered tainted, along with anything it touches. That means that `$quantity` should be considered untrustworthy, along with `$total` and `$totalString`, since they get their values (in part) from a tainted source.

Of course, it is easy to say which values are clean and which are dirty in this small example. But in a larger application, one which has many source files and parts interacting, it becomes extremely hard to trace the cleanliness of any given variable. For this reason, it makes sense to simply sanitize everything: use prepared statements for all database inputs, and call `htmlentities()` on all outputs. Ultimately, it's better to be safe than sorry.

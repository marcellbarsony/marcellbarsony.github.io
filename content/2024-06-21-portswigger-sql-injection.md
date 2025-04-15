+++
title = "SQL injection"
date = 2024-06-21

[taxonomies]
tags = ["webapp", "portswigger", "burp-suite", "server-side", "sql injection"]
+++

![sql-injection](/pictures/articles/portswigger/sql-injection/sql-injection.svg)

**SQL injection (SQLi)** is a vulnerability that allows an attacker to interfere
with the queries that an application makes to its database. This might allow an
attacker to view, modify or delete sensitive data they are not authorized to
access.

<!-- more -->

SQL injections can be detected manually using test patterns against application
entry points:
- `'` - Single quote character
- `OR 1=1` and `OR 1=2` - Boolean conditions
- Payloads to trigger time delays when executed

## Exploitation

<!-- LAB 1 {{{-->
### [LAB 1 - SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/sql-injection-apprentice/sql-injection/lab-retrieve-hidden-data)

With the filters on the webshop's header, it is possible to narrow down
the list of products currently offered.

![lab1-1](/pictures/articles/portswigger/sql-injection/lab1-1.png)

The request header  `GET /filter?category=Pets HTTP/2` may be vulnerable
to SQL injection if the category parameter is directly used in an SQL query
without proper sanitization or parameterization.

![lab1-2](/pictures/articles/portswigger/sql-injection/lab1-2.png)

Giving it the value `'+OR_1=1` the product category filter can be
modified to inject an SQL query that evaluates to `true` and comments out
(disregards) whatever is following the statement.

![lab1-3](/pictures/articles/portswigger/sql-injection/lab1-3.png)

- `'` - Closes a string prematurely
- `+` - Includes the rest of the injection
- `OR` - Combines multiple conditions in a `WHERE` clause
- `1=1` - Evaluates to `true`
- `--` - Disregards the rest of the query
<!-- }}} -->

<!-- LAB 2 {{{-->
### [LAB 2 - SQL injection vulnerability allowing login bypass](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/sql-injection-apprentice/sql-injection/lab-login-bypass)

<!-- }}} -->

<!-- Mitigation {{{-->
## Mitigation

<!-- }}} -->

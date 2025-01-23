+++
title = "Server-Side Request Forgery [SSRF]"
date = 2024-06-17

[taxonomies]
tags = ["webapp", "portswigger", "burp-suite", "server-side", "ssrf" ]
+++


![ssrf](/pictures/articles/server-side-request-forgery/server-side-request-forgery.svg)

**Server-Side Request Forgery (SSRF)** allows an attacker to cause the
server-side application to make an HTTP requests to an unintended, arbitrary
(internal or external) location. SSRF vulnerabilities currently hold the #10
spot on the [OWASP Top 10 list](https://owasp.org/www-project-top-ten/)
(as of 2021).

<!-- more -->

Applications may be vulnerable to SSRF due to the following reasons:
- The access control check might be implemented in a component that sits in
  front of the server.
- The application might allow administrative access without logging in to any
  user coming from a local machine: this assumes that only a trusted user would
  come directly from the server.
- The administrative interface might listen to the main application on a
  different port: this might not be reachable by regular users.

## Exploitation

<!-- LAB 1 {{{ -->
### [LAB 1 - Basic SSRF against the local server](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/ssrf-apprentice/ssrf/lab-basic-ssrf-against-localhost)

In this web app, we can utilize the `Check stock` feature to gather information
about the availability of a product (which is a Giant Enter Key in this case -
very nice - however, I prefer the ANSI layout over the ISO).

![ssrf](/pictures/articles/server-side-request-forgery/ssrf-1.png)

By clicking on `Check stock` and intercepting the request with Burp Suite's
proxy, we can notice that a `stockApi` request is being made to an encoded URL.

![ssrf](/pictures/articles/server-side-request-forgery/ssrf-2.png)

Replacing this URL with `http://localhost/`, we can find ourselves on a hidden
Admin panel however, we cannot perform any privileged actions here just yet.
Once we hove over `Delete` next to Carlos's name we can observe the URL in the
status bar.

![ssrf](/pictures/articles/server-side-request-forgery/ssrf-3.png)

We can now send this request to Burp Suite's Repeater and modify it to be
`https://localhost/admin/delete?username=carlos`. We receive a [302 Found](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/302)
response which indicates a redirection to another location which can be followed
with the `Follow redirection` button.

![ssrf](/pictures/articles/server-side-request-forgery/ssrf-4.png)

Finally, we receive a [401 Unauthorized](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401)
status code because we're trying to reach the admin page from an outside
perspective.

![ssrf](/pictures/articles/server-side-request-forgery/ssrf-5.png)

Despite the error, the lab is now completed as Carlos's account has been
deleted.

<!-- }}} -->

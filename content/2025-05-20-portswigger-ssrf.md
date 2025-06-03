+++
title = "Portswigger - Server-Side Request Forgery [SSRF]"
date = 2025-05-20

[taxonomies]
tags = ["webapp", "portswigger", "burp suite", "server-side", "ssrf" ]
+++

![ssrf](/pictures/articles/portswigger/server-side-request-forgery/server-side-request-forgery.svg)

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

<!-- LAB 1 {{{-->
### [LAB 1 - Basic SSRF against the local server](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/ssrf-apprentice/ssrf/lab-basic-ssrf-against-localhost)

In this web app, the `Check stock` feature can be utilized to gather information
about the availability of a product (which is a Giant Enter Key in this case -
very nice, however, I prefer the ANSI layout over the ISO).

![ssrf](/pictures/articles/portswigger/server-side-request-forgery/lab-1-1.png)

By clicking on `Check stock` and intercepting the request with Burp Suite's
proxy reveals that a `stockApi` request is being made to an encoded URL.

![ssrf](/pictures/articles/portswigger/server-side-request-forgery/lab-1-2.png)

Replacing this URL with `http://localhost/`, the request can be redirected
from its intended location to a hidden Admin panel however,
privileged actions cannot be performed just yet. Hovering over
`Delete` next to `carlos`'s name the URL in the status bar can be observed.

![ssrf](/pictures/articles/portswigger/server-side-request-forgery/lab-1-3.png)

Request can now be sent to Burp Suite's Repeater and modified to be
`https://localhost/admin/delete?username=carlos`. A [302 Found](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/302)
response is returned, which indicates a redirection to another location which
can be followed with the `Follow redirection` button.

![ssrf](/pictures/articles/portswigger/server-side-request-forgery/lab-1-4.png)

The process concludes with a [401 Unauthorized](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401)
status code, as access to the admin page is being attempted from an external
perspective.

![ssrf](/pictures/articles/portswigger/server-side-request-forgery/lab-1-5.png)

Despite the error, the lab is now completed as `carlos`'s account has been
deleted.
<!-- }}} -->

<!-- LAB 2 {{{-->
### [LAB 2 - SSRF attacks against other back-end systems](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/ssrf-apprentice/ssrf/ssrf-attacks-against-other-back-end-systems)

Once again, the stock-checking feature is retrieving product availability data
from the backend. This `stockApi` request can be intercepted and modified
using Burp Suite.

![ssrf](/pictures/articles/portswigger/server-side-request-forgery/lab-2-1.png)

The well-known `192.168.0.x` private IP range (`1-255`) can be scanned with
Burp Suite's Intruder to potentially discover other internal functionalities.

![ssrf](/pictures/articles/portswigger/server-side-request-forgery/lab-2-2.png)

The [200 OK](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200)
status code indicates a successful response from `192.168.0.85:8080/admin`.

![ssrf](/pictures/articles/portswigger/server-side-request-forgery/lab-2-3.png)

Replacing `stockApi` with the found address reveals the hidden administrator
panel.

![ssrf](/pictures/articles/portswigger/server-side-request-forgery/lab-2-4.png)

The lab now can be solved by making a request to
`http://192.169.0.85:8080/admin/delete?username=carlos` that removes the user
`carlos`.

<!-- }}} -->

## Mitigation

To avoid **Server-Side Request Forgery** vulnerabilities,
developers should implement the following security measures:

1. Input Validation & Allowlist
    - Only allow requests to explicitly approved domains or IPs
    - Block user input from directly controlling request URLs

2. Network Restrictions:
    - Restrict outbound requests from the server to internal or private networks
    - Use firewall rules or network policies to prevent access
      to internal services

3. URL Parsing & Validation:
    - Parse and normalize URLs before making requests
      to avoid bypass techniques

4. Enforce Least Privilege:
    - Run services with minimal network permissions
      to prevent unauthorized internal access

5. Monitor & Log Requests:
    - Implement logging and anomaly detection for unusual outbound requests

+++
title = "Portswigger - Directory traversal vulnerabilities"
date = 2025-05-21

[taxonomies]
tags = ["webapp", "portswigger", "burp-suite", "server-side", "directory-traversal", "path-traversal"]
+++

![directory-traversal](/pictures/articles/portswigger/directory-traversal/directory-traversal.svg)

**Directory traversal** (or path traversal) vulnerabilities enable attackers to read
arbitrary files (e.g.: application source code, credentials and other  other
sensitive data) on the web application server. These vulnerabilities exist when
applications don't handle user-supplied input properly in the file paths. By
modifying the client's request, attackers can trick the web application into
accessing files outside its intended location by basically hitchhiking through
directories.


<!-- more -->


## Exploitation

<!-- LAB 1 {{{-->
### [LAB 1 - File path traversal, simple case](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/path-traversal-apprentice/file-path-traversal/lab-simple)

The example web shop application loads an image using the following HTML code:
```html
<img src="/loadImage?filename=21.png">
```

The `loadImage` URL takes a `filename` parameter (`21.png`) and returns the
contents of the specified file.

With Burp Suite this [HTTP GET request](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/GET)
can be intercepted and modified to retrieve the `/etc/passwd` file from the
server.

![directory-traversal_burp](/pictures/articles/portswigger/directory-traversal/directory-traversal_request.png)

The HTTP response includes the contents of the aforementioned file that is
storing essential information about the user accounts utilized on the server
such as
- the username,
- user ID,
- group ID,
- home directory,
- and default shell.

![directory-traversal_response_burp](/pictures/articles/portswigger/directory-traversal/directory-traversal_response.png)
<!--}}}-->

## Mitigation

<!-- Mitigation {{{-->
To mitigate such vulnerabilities, **developers should never blindly trust
user-supplied inputs** that could potentially interact with the file system API,
and must implement robust validation techniques such as
- input sanitizing,
- directory whitelisting,
- path canonicalization,
- or limiting permissions (least privilege principle).
<!-- }}} -->

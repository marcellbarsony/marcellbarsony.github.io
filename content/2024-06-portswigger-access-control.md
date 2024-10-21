+++
title = "Access control vulnerabilities"
date = 2024-06-15

[taxonomies]
tags = ["webapp", "portswigger", "burp-suite", "server-side", "access-control"]
+++


![access-control](/pictures/articles/access-control/access-control.svg)

<!-- Access control is the application of constraints on who or what is -->
<!-- authorized to perform actions or access resources. -->
<!-- In the context of web applications, access control is dependent on -->
<!-- authentication and session management: -->

- **Authentication** confirms that the user is who they say they are
- **Session management** identifies if HTTP requests are made by the same user
- **Access control** determines whether the user is authorized to carry out the
  action that they are attempting to perform.

<!-- Design and management of access controls is a complex and dynamic -->
<!-- problem that applies business, organizational, -->
<!-- and legal constraints to a technical implementation. -->


<!-- more -->


## Exploitation

<!-- LAB 1 {{{ -->
### [LAB 1 - Unprotected admin functionality](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/lab-unprotected-admin-functionality)

By appending `/robots.txt` to the end of the lab URL we can read the
content of the file.

![access-control](/pictures/articles/access-control/lab1.png)

`robots.txt` implements the [Robots Exlusion Standard](https://en.wikipedia.org/wiki/Robots.txt)
([RFC 9309](https://www.rfc-editor.org/rfc/rfc9309.html)) by telling the
search engine crawlers which URLs they can and can't access.

In this case, the hidden and unprotected administrator panel is being exposed
as `robots.txt` cannot be used to safeguard critical website functionality.
<!-- }}} -->

<!-- LAB 2 {{{ -->
### [LAB 2 - Unprotected admin functionality with unpredictable URL](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/lab-unprotected-admin-functionality-with-unpredictable-url)

Some applications try to protect sensitive features with non-obvious URLs
however, this approach, known as "security by obscurity", is not secure.

![access-control](/pictures/articles/access-control/lab2.png)

The webpage's source code contains a script that adds a link to the UI if the
user is an administrator.

By appending `/admin-kozui4` to the end of the lab URL we can access the hidden
and unprotected administrator panel.
<!-- }}} -->

<!-- LAB 3 {{{ -->
### [LAB 3 - User role controlled by request parameter](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/lab-user-role-controlled-by-request-parameter)

Applications may determine the user's access rights at login,
and then store this information in a user-controllable location, that could be:

```sh
# Hidden field
<input type="hidden" name="access_level" value="admin">

# Cookie
Cookie: Admin=true

# Preset query string parameter
https://insecure-website.com/login/home.jsp?admin=true
```

Accessing the lab's `/admin` panel, the HTTP GET request sets the `Admin=false`
cookie.

![access-control-request-parameter](/pictures/articles/access-control/lab3.png)

Setting the cookie's value to `true` gives us access to the admin panel.
<!-- }}} -->

<!-- LAB 4 {{{ -->
### [LAB 4 - User ID controlled by request parameter, with unpredictable user IDs](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/lab-user-id-controlled-by-request-parameter-with-unpredictable-user-ids)

While browsing the example blog we can stumble upon our victim's blog post
titled "_Identity Theft_".

Clicking on Carlos's username we can intercept their user GUID parameter
from the HTTP GET request.
![access-control-request-parameter](/pictures/articles/access-control/lab4-1.png)

By modifying the user GUID sent to the server during our own login,
we can gain access to Carlos's account without knowing their credentials.
![access-control-request-parameter](/pictures/articles/access-control/lab4-2.png)

This process results in a horizontal privilege escalation, compromising the
victim's secret API key.
![access-control-request-parameter](/pictures/articles/access-control/lab4-3.png)
<!-- }}} -->

<!-- LAB 5 {{{ -->
### [LAB 5 - User ID controlled by request parameter with password disclosure](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/lab-user-id-controlled-by-request-parameter-with-password-disclosure)

After logging in to our user account, we can modify the `id` parameter in the
URL from our username to `administrator`:<br>
`/my-account?id=administrator`

Capturing the HTTP response we can notice it contains an input field pre-filled
with the administrator's password in clear text.
![access-control-request-parameter](/pictures/articles/access-control/lab5.png)
<!-- }}} -->
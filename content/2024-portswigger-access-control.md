+++
title = "Access control vulnerabilities"
date = 2024-03-21

[taxonomies]
tags = ["portswigger", "burp-suite", "access-control"]
+++


![access-control](/pictures/access-control.svg)


<!-- Access control is the application of constraints on who or what is -->
<!-- authorized to perform actions or access resources. -->
<!-- In the context of web applications, access control is dependent on -->
<!-- authentication and session management: -->

- **Authentication** confirms that the user is who they say they are
- **Session management** identifies if HTTP requests are made by the same user
- **Access control** determines whether the user is authorized to carry
out the action that they are attempting to perform.

<!-- Design and management of access controls is a complex and dynamic -->
<!-- problem that applies business, organizational, -->
<!-- and legal constraints to a technical implementation. -->


<!-- more -->


## Exploitation

**LAB 1 - [Unprotected admin functionality](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/lab-unprotected-admin-functionality)**

By appending `/robots.txt` to the end of the lab URL we can read the
contents of the file.

![access-control](/pictures/access-control-robots.png)

`robots.txt` implements the Robots Exlusion Standard by telling the
search engine crawlers which URLs they can and can't access.

In this case the hidden and unprotected administrator panel is being exposed.

**LAB 2 - [Unprotected admin functionality with unpredictable URL](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/lab-unprotected-admin-functionality-with-unpredictable-url)**

Some applications try to protect sensitive features with non-obvious URLs,
this approach, known as "security by obscurity", is not secure.

![access-control-unprotected-admin](/pictures/access-control-unprotected-admin.png)

The webpage's source code contains a script that adds a link to the UI if the
user is an administrator.

### LAB 3 - User role controlled by request parameter

Some applications determine the user's access rights or role at login,
and then store this information in a user-controllable location that could be:

- A hidden field
- A cookie
- A preset query string parameter


### LAB 4 - User ID controlled by request parameter, with unpredictable user IDs

### LAB 5 - User ID controlled by request parameter with password disclosure

## Mitigation



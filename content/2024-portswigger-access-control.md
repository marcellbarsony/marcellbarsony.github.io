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

### LAB 1 - Unprotected admin functionality

By appending `/robots.txt` to the end of the lab URL we can read the
contents of the file.

![access-control](/pictures/access-control-robots.png)

`robots.txt` implements the Robots Exlusion Standard by telling the
search engine crawlers which URLs they can and can't access.

In this case the hidden unprotected administrator panel is being exposed.

### LAB 2 - Unprotected admin functionality with unpredictable URL

### LAB 3 - User role controlled by request parameter

### LAB 4 - User ID controlled by request parameter, with unpredictable user IDs

### LAB 5 - User ID controlled by request parameter with password disclosure

## Mitigation



+++
title = "Authentication vulnerabilities"
date = 2024-06-16

[taxonomies]
tags = ["webapp", "portswigger", "burp-suite", "server-side", "authentication",
"mfa"]
+++

![authentication-vulnerabilities](/pictures/articles/portswigger/authentication-vulnerabilities/password-reset-poisoning.svg)

**Authentication** is the process of verifying that a user is who they claim to
be. Authentication vulnerabilities can allow attackers to gain access to
sensitive data or functionality. They also expose additional attack surface for
further exploits.

<!-- more -->

## Brute-force attacks

A brute-force attack is when an attacker uses a system of trial and error to
guess valid user credentials. These attacks are typically automated using
word lists of usernames and passwords. Automating this process, especially using
dedicated tools, potentially enables an attacker to make vast numbers of login
attempts at high speed.

## Exploitation

### [LAB 1 - Username enumerating via different responses](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/authentication-apprentice/authentication/password-based/lab-username-enumeration-via-different-responses)



![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-1-1.png)

The captured request then can be sent to the Intruder to brute-force
the credentials.

![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-1-2.png)

The username parameter can be set as a payload position by enclosing it in `§`.

![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-1-3.png)

The payload can then be configured using a word list that Burp Suite's Intruder
can iterate through. The word list in this case contains possible usernames.

![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-1-4.png)

The difference in response length suggests that one of the usernames is
returning a different response: `Incorrect password`

![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-1-5.png)

A similar approach can be used to identify the corresponding password.

![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-1-6.png)

The response status code [302 Found](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/302)
indicates that the login attempt was successful.

![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-1-7.png)

It is now possible to log in to the account with the username `americas`
and password `football`.

![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-1-8.png)

### [LAB 2 - 2FA simple bypass](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/authentication-apprentice/authentication/multi-factor/lab-2fa-simple-bypass)


## Mitigation

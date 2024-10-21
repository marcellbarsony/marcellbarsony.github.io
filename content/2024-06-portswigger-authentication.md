+++
title = "Authentication vulnerabilities"
date = 2024-06-17

[taxonomies]
tags = ["webapp", "portswigger", "burp-suite", "server-side", "authentication",
"mfa"]
+++


![authentication-vulnerabilities](/pictures/articles/authentication-vulnerabilities/password-reset-poisoning.svg)

**Authentication** is the process of verifying that a user is who they claim to be.
Authentication vulnerabilities can allow attackers to gain access to sensitive
data or functionality. They also expose additional attack surface for further
exploits.

<!-- more -->

## Brute-force attacks

A brute-force attack is when an attacker uses a system of trial and error to
guess valid user credentials. These attacks are typically automated using
wordlists of usernames and passwords. Automating this process, especially using
dedicated tools, potentially enables an attacker to make vast numbers of login
attempts at high speed.

## Exploitation


### [LAB 1 - Username enumeratin via different responses](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/authentication-apprentice/authentication/password-based/lab-username-enumeration-via-different-responses)


### [LAB 2 - 2FA simple bypass](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/authentication-apprentice/authentication/multi-factor/lab-2fa-simple-bypass)

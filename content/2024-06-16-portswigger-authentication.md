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

<!-- LAB 1 {{{-->
### [LAB 1 - Username enumerating via different responses](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/authentication-apprentice/authentication/password-based/lab-username-enumeration-via-different-responses)

The first step involves identifying how the login form works
by capturing the HTTP request that is sent when credentials are submitted.

![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-1-1.png)

The captured request then can be sent to the Intruder tool to perform
a brute-force attack on the login funcitonality.

![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-1-2.png)

To target the `username` field, the parameter can be marked as a payload
position by surrounding it with `§` symbols.

![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-1-3.png)


Next, a payload list is configured. In this case, a list of potential usernames
is loaded into Intruder. The tool will iterate through each value and inject it
into the defined payload position.

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
<!--}}}-->

<!-- LAB 2 {{{-->
### [LAB 2 - 2FA simple bypass](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/authentication-apprentice/authentication/multi-factor/lab-2fa-simple-bypass)

Accessing the provided user account (`wiener:peter`) allows for exploration
and analysis of the web application’s functionality.

![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-2-1.png)

After providing with the credentials, the system sends an e-mail
containing a second-factor authentication code.

![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-2-2.png)

This code must be entered to complete the two-factor authentication process.

![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-2-3.png)

The URL `/my-account?id=wiener` is displayed after authentication,
which indicates that the account is identified via a query parameter.<br>
This behavior can potentially be leveraged to bypass
the second-factor authentication.

![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-2-4.png)

By changing the URL to `/my-account?id=carlos` it is possible to gain
unauthorized access to `carlos`'s account as the web application does not
implement proper checks for the second-factor authentication.

![auth](/pictures/articles/portswigger/authentication-vulnerabilities/lab-2-5.png)
<!--}}}-->

## Mitigation

<!-- Mitigation {{{-->
1. Implement strong password policies
    - Enforce password complexity and length requirements
    - Check against commonly used/breached passwords

2. Multi-factor authentication
    - Implement additional verification methods beyond passwords
    - Include multiple options: Authenticator APP, biometrics, security keys

3. Session management
    - Generate secure, random session IDs
    - Set appropriate timeouts and invalidate on logout

4. Rate limiting and account lockout
    - Limit failed login attempts
    - Implement progressive delays between attempts

5. Secure password storage
    - Never store plain text passwords
    - Use strong and modern hashing algorithms with proper salting

6. HTTPS everywhere
    - Encrypt all traffic to prevent credential interception
    - Implement strict transport security headers
<!-- }}} -->

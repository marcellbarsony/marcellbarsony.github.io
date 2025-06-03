+++
title = "Portswigger - OS command injection"
date = 2025-05-22

[taxonomies]
tags = ["webapp", "portswigger", "burp suite", "server-side", "os command injection"]
+++

![os-command-injection](/pictures/articles/portswigger/os-command-injection/os-command-injection.svg)

**OS command injection** (also known as shell injection) allows an attacker to
manipulate user input to execute arbitrary operating system commands on the
server running the web application. This often occurs when an application
improperly passes unsanitized to system functions like `exec()`, `system()`,
`popen()`, or relies on shell commands for functionality
without proper input validation.


<!-- more -->


## Exploitation

<!-- LAB 1 {{{-->
### [LAB 1 - OS command injection, simple case](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/os-command-injection-apprentice/os-command-injection/lab-simple)

The Giant Enter Key is back again! With the `Check stock` feature,
it is possible to query the webshop's stock information.

![os-command-injection](/pictures/articles/portswigger/os-command-injection/lab-1-1.png)

Burp Suite's proxy can intercept this user-controllable request:
the `storeId` parameter appears to be vulnerable to shell injection,
allowing modification to execute arbitrary commands on the server's
operating system. By appending the
[whoami](https://en.wikipedia.org/wiki/Whoami) command, we can determine
the user context in which the server executes commands.

![os-command-injection](/pictures/articles/portswigger/os-command-injection/lab-1-2.png)

The reason behind why the pipe (`|`) symbol should be used instead of the
ampersand (`&`) is that `whoami` doesn't accept anything in standard input yet
still executes, even if the command beforehand fails or has no meaningful
output.
<!-- }}} -->

## Mitigation

<!-- Mitigation {{{-->
1. Avoid Directly Executing User Input
- Use safer alternatives like built-in APIs instead of system commands

2. Use Parameterized Commands
- Implement safe command execution methods (e.g., subprocess.run([...], check=True)
  in Python with arguments as lists).

3. Input Validation & Allowlisting:
- Accept only predefined and expected input values;
  reject special characters (; | & $).

4. Least Privilege Principle
- Run applications with minimal system privileges to reduce the impact of exploitation.

5. Disable Unnecessary System Commands
- Restrict access to dangerous shell utilities (e.g., sh, bash, cmd.exe).

6. Use Web Application Firewalls (WAFs)
- Deploy WAFs to detect and block suspicious input patterns.
<!-- }}} -->

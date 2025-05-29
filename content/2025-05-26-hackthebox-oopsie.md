+++
title = "Hack The Box - Oopsie"
date = 2025-05-29

[taxonomies]
tags = ["hackthebox", "idor", "reverse-shell", "suid", "privesc"]
+++

[![TODO](/pictures/articles/htb/archetype/cover.png)](https://labs.hackthebox.com/achievement/machine/447801/287)


**Oopsie** is a Linux (Ubuntu) box created to teach the impact of Information
Disclosure and [Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
vulnerabilities and chain together multiple vulnerabilities to escalate
privileges on the target system.


<!-- more -->


## Enumeration

<!-- Enumeration {{{-->

The [nmap](https://nmap.org/) scan shows that the target has port 22
[SSH](https://en.wikipedia.org/wiki/Secure_Shell) open and is running a HTTP
(web) server on port 80.

![nmap-scan](/pictures/articles/htb/oopsie/01-nmap.png)

- `-sC`: Script scan
- `-sV`: Version detection

The web server serves a webpage however, it's just a static website that doesn't
have much functionality.

![webpage](/pictures/articles/htb/oopsie/02-webpage.png)

After setting up a proxy with Burp Suite, the site map reveals that there is a
hidden login page at `/cdn-cgi/login`.

![site-map](/pictures/articles/htb/oopsie/03-site-map.png)

The username and the corresponding password is unknown, but the site has an
option to log in as Guest.

![test](/pictures/articles/htb/oopsie/04-login-page.png)

After logging in, Burp Suite intercepts a cookie where the user ID is set to
`2233` and the role assigned to the session is `guest`.

![test](/pictures/articles/htb/oopsie/05-login-cookie.png)

On the website, there is an option to upload some files, but it requires super
admin rights.

![test](/pictures/articles/htb/oopsie/06-uploads.png)

Also there is an `Account` option to check the details
of the currently logged in session.

![test](/pictures/articles/htb/oopsie/07-account.png)

In the URL bar, there is an `id` parameter with the value of `2`. This value can
be modified to `1` that leads to an information disclosure vulnerability
by revealing the Access ID (`34322`) of the `admin` account.

![test](/pictures/articles/htb/oopsie/08-account.png)

The cookie stored in the browser for the `guest` user can now be modified to
represent the ID (`34322`) discovered for the `admin` account - _the role
`guest` could also be modified to `admin` however, this seems optional in this
lab_.

![test](/pictures/articles/htb/oopsie/09-cookie-storage.png)

The upload functionality is now accessible, which opens the possibility of
uploading a reverse shell that could potentially be exploited.

![test](/pictures/articles/htb/oopsie/10-uploads.png)

<!-- }}} -->

## Reverse shell

<!-- Reverse shell {{{-->

Let's modify a simple PHP reverse shell so that it will initiate a connection
back to the attacker machine (`10.10.15.124`).

![test](/pictures/articles/htb/oopsie/11-revshell.png)

The upload of the reverse shell script was successful, but unfortunately,
there's no indication of where the script was uploaded.

![test](/pictures/articles/htb/oopsie/12-uploaded.png)

[Gobuster](https://github.com/OJ/gobuster) could be used to brute-force
directories on the site using a word list, and it found a directory named
`/uploads`, which has now become a point of interest.

![test](/pictures/articles/htb/oopsie/13-gobuster.png)

Prior to calling the reverse shell script in `/uploads`, a
[netcat](https://hu.wikipedia.org/wiki/Netcat) must be set up to listen to the
incoming connection.

![test](/pictures/articles/htb/oopsie/14-netcat.png)

- `l`: Listen mode
- `v`: Verbose mode
- `n`: No DNS resolution
- `p`: Port number

The reverse shell can be triggered by accessing its URL on the web server.

![test](/pictures/articles/htb/oopsie/15-query.png)

Netcat has received the incoming connection and spawned a simple shell.

![test](/pictures/articles/htb/oopsie/16-connection.png)

Since [Python](https://www.python.org/) is installed on the target, it can be
utilized to create an interactive shell.

![test](/pictures/articles/htb/oopsie/17-interactive-shell.png)

- `c`: Run the following Python code in the command line
- `import pty`: Import the pseudo-terminal module
- `pty.spawn("/bin/bash")`: Spawn a new process (a Bash shell) and connect it to
  the pseudo-terminal for an interactive session

<!-- }}} -->

## Lateral movement

<!-- Lateral movement {{{-->

As the user `www-data` it is not possible to achieve many things,
so either lateral movement or a privilege escalation is needed
to further exploit the system.

The root directory of the web server (`/var/www/`) can be investigated for
plain-text passwords by [grepping](https://en.wikipedia.org/wiki/Grep) them for
the string `passw`.
 
![test](/pictures/articles/htb/oopsie/18-plain-text-password.png)

After finding a plain-text password for the `admin` user, the `/etc/passwd` file
should be searched to identify existing users on the system.

![test](/pictures/articles/htb/oopsie/19-users.png)

Unfortunately, the found password doesn't work for the user named `robert`.

![test](/pictures/articles/htb/oopsie/20-failed-attempt.png)

The script named `db.php` in fact contains `robert`'s real password.

![test](/pictures/articles/htb/oopsie/21-authentication.png)

The user flag can be found in `robert`'s home directory, in the file `user.txt`.

![test](/pictures/articles/htb/oopsie/22-user-flag.png)

<!-- }}} -->

# Privilege Escalation

<!-- Privilege Escalation {{{-->

Unfortunately, `robert` is not a member of the `wheel` group so that this
account cannot execute commands as sudo.

![test](/pictures/articles/htb/oopsie/23-privesc-fail.png)

- `-l`: List the user's allowed commands and privileges

However, the `id` command shows that `robert` is the member of the `bugtracker`
group which could be investigated further.

![test](/pictures/articles/htb/oopsie/24-privesc-enum.png)

The file system can be searched for files belonging to the group `bugtracker`.

![test](/pictures/articles/htb/oopsie/25-privesc.png)

- `/`: Search the root directory
- `-group`: Find files belonging to the specified group

The found file can be enumerated with the `ls -al` and the `file` commands.

The `file` command reveals that there is a `suid` set on the found binary.

SUID (Set owner User ID) is a special permission: A file with SUID set always
executes as the user who owns the file (`root`), regardless of which user is
issuing the command.

![test](/pictures/articles/htb/oopsie/26-privesc-enum-file.png)

The binary is accepts user input as a filename which contents will be dumped
using the `cat` command.

![test](/pictures/articles/htb/oopsie/27-privesc-run-file.png)

Creating a file named `cat` in the `/tmp` directory with the content `/bin/bash`
and making it executable with the `chmod +x cat` command will launch a bash
shell upon execution.

![test](/pictures/articles/htb/oopsie/28.png)

To launch the exploit, the `/tmp` directory needs to be added to the `PATH`
environmental variable.

The `PATH` environment variable is a list of directories that the shell searches
through to find the corresponding executable file.

![test](/pictures/articles/htb/oopsie/29-privesc-path.png)

Launching `bugtracker` from the `/tmp` directory will spawn a root shell and
escalate privileges on the system.

![test](/pictures/articles/htb/oopsie/30-privesc.png)

The `root` flag can be found in the `/root/root.txt`.

![test](/pictures/articles/htb/oopsie/31-root-flag-open.png)

The contents of the file cannot be viewed with `cat` so I opened it with
[vim](https://www.vim.org/).

![test](/pictures/articles/htb/oopsie/32-root-flag.png)

By obtaining the root flag the machine is
[pwned](https://www.hackthebox.com/achievement/machine/447801/288)

<!-- }}} -->

# Mitigation

<!-- Mitigation {{{-->

1. Hidden login page
    - The hidden login page should be protected with additional layers of
    security (VPN, IP whitelisting, etc.)

2. Weak session management
    - Session cookies can be intercepted and modified to manipulate user roles
    and gain unauthorized access
    - Use proper session management mechanisms (regenerate session IDs upon
    login or role change)
    - Sanitize and validate user inputs, including URL parameters and cookies
    - Use secure cookies with the `Secure` and `HttpOnly` flags

3. File upload vulnerability
    - Validate uploaded files and only allow the required file types
    - Store uploaded files outside the web root or disable execution scripts in
    upload directories
    - Enforce antivirus scanning on uploaded files

4. Directory brute-force
    - Ensure directory indexing is disabled
    - Use a WAF (Web Application Firewall) to block or rate-limit brute-force
    attempts

5. Insecure reverse shell trigger
    - Ensure internal networks are segmented from production systems
    - Restrict outbound traffic for web servers to prevent reverse shell
    connections

6. Plain-text password disclosure
    - Avoid storing clear-text password and use strong encryption algorithms to
    hash passwords
    - Use secrets management to securely store passwords

7. SUID binary exploit
    - Minimize the use of SUID binaries
    - Monitor files with SUID set to ensure they are necessary and secure
    - Use file integrity monitoring to detect changes to sensitive binaries

8. Path manipulation
    - Do not allow users to modify system environmental variables
    - Limit the set of commands a user can run with sudo

<!-- }}} -->

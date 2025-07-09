+++
title = "TryHackMe - Dreaming"
date = 2025-06-24

[taxonomies]
tags = ["tryhackme", "file-upload", "privesc", "reverse-shell", "sql"]
+++

![TODO](/pictures/articles/thm/dreaming/00-cover.png)

**Dreaming** - inspired by *The Sandman* comic book - is a Pluck CMS-based
machine that shows how weak credentials and an unpatched file upload
vulnerability can be chained to gain remote command execution. It also
highlights how improper sudo permissions and insecure coding practices - such as
storing plaintext credentials - can lead to privilege escalation and full system
compromise.

<!-- more -->

## Enumeration

<!-- Enumeration {{{-->

Nmap shows that the following ports are open on the target machine:

- 22 - OpenSSH
- 80 - Apache HTTP Server

![a](/pictures/articles/thm/dreaming/01-nmap.png)

The site hosted on port `80` only displays the default Apache 2 page.

![a](/pictures/articles/thm/dreaming/02-default-page.png)

Fuzzing directories with gobuster results in a hit that can be further
investigated.

![a](/pictures/articles/thm/dreaming/03-gobuster.png)

The directory structure reveals the CMS used on the web application.

![a](/pictures/articles/thm/dreaming/04-webapp.png)

Searching for information on the version number of the CMS leads to a
**file upload remote code execution** vulnerability
([CVE-202029607](https://www.exploit-db.com/exploits/49909)).

![a](/pictures/articles/thm/dreaming/05-dreaming.png)

<!-- }}} -->

# Exploitation

<!-- Exploitation {{{-->

<!-- Lucien {{{-->
## Lucien

Clicking on `admin` directs to the Pluck login page, but the password is
currently unknown.

![a](/pictures/articles/thm/dreaming/06-login.png)

After some guessing, it is possible to gain access to the administrator panel
with the password `password`.

![a](/pictures/articles/thm/dreaming/07-pluck.png)

To leverage the aforementioned CVE, a reverse-shell can be uploaded to initiate
a connection back to the attacker machine once called with the magnifying glass
icon.

![a](/pictures/articles/thm/dreaming/08-file-upload.png)

The reverse-shell connection is captured by netcat on port `1234` as expected.

![a](/pictures/articles/thm/dreaming/09-reverse-shell.png)

Once the connection is established, the spawned shell can be upgraded to an
interactive shell.

![a](/pictures/articles/thm/dreaming/10-interactive-shell.png)

Investigating the `/etc/passwd` file reveals three users of interest: `lucien`,
`death` and `morpheus`.

![a](/pictures/articles/thm/dreaming/11-whoami.png)

To enumerate the machine further [LinPEAS](https://github.com/peass-ng/PEASS-ng)
can be utilized.

![a](/pictures/articles/thm/dreaming/12-linpeas.png)

LinPEAS has found a couple unexpected files in the `/root` directory.

![a](/pictures/articles/thm/dreaming/13-linpeas-root.png)

A local instance of MySQL is running on port `3306`.

![a](/pictures/articles/thm/dreaming/14-linpeas-ports.png)

![a](/pictures/articles/thm/dreaming/15-linpeas-users.png)

In the web server's directory, there is an interesting file called `pass.php`
which contains a string of random characters that looks like a hash.

![a](/pictures/articles/thm/dreaming/16-hash.png)

After checking it with `hashid`, it appears to be a `SHA-512` hash.

![a](/pictures/articles/thm/dreaming/17-hash-id.png)

Running it through `hashcat` against a wordlist uncovers the password to be
`password`.

![a](/pictures/articles/thm/dreaming/18-hash-crack.png)

The `/opt` directory also contains some unusual files that should be
investigated further.

![a](/pictures/articles/thm/dreaming/19-opts.png)

`test.py` discloses `lucien`'s password in plain text.

![a](/pictures/articles/thm/dreaming/20-lucien-pass.png)

It is now possible to change the current user and authenticate as `lucien`.

![a](/pictures/articles/thm/dreaming/21-su-lucien.png)

The file `lucien_flag.txt` in the user's home directory contains the first flag.

![a](/pictures/articles/thm/dreaming/22-lucien-flag.png)

<!-- }}} -->

<!-- Death {{{-->
## Death

The command `sudo -l` shows that `lucien` should be able to run `getDreams.py`
from `death`'s home directory.

![a](/pictures/articles/thm/dreaming/23-sudo-l.png)

It is indeed possible to run the Python script, but its behavior cannot be
leveraged just yet.

![a](/pictures/articles/thm/dreaming/24-getdreams.png)

`lucien`'s `.bash-history` file discloses a MySQL password.

![a](/pictures/articles/thm/dreaming/25-bash-history.png)

![a](/pictures/articles/thm/dreaming/26-bash-history-2.png)

The found passsword can be used to authenticate as `lucien`to the local MySQL
instance running on the server.

![a](/pictures/articles/thm/dreaming/27-mysql.png)

The database shows the same records as the output from the previously run
script.

![a](/pictures/articles/thm/dreaming/28-mysql-show.png)

It is possible to inject a custom record to copy an instance of `/bin/bash` to a
temporary directory and make it executable.

![a](/pictures/articles/thm/dreaming/29-mysql-insert.png)

When the script is executed again from `death`'s directory, it spawns a shell
under the name of `death`.

![a](/pictures/articles/thm/dreaming/30-getdreams.png)

`death`'s flag can be found in `death_flag.txt` in its home directory.

![a](/pictures/articles/thm/dreaming/31-death-flag.png)

<!-- }}} -->

<!-- }}} -->

## Mitigation

<!-- Mitigation {{{-->

1. Apply patches to known CVEs
    - Regularly audit, update and patch critical CMS platforms such as Pluck

2. Sanitize file uploads
    - Implement strict file type validation, remove executable permissions on
    uploaded files, and store uploads outside the web root

3. Restrict sudo permissions
    - Limit the use of `sudo` to essential commands, and avoid allowing users to
    execute scripts owned by higher-privileged accounts

4. Secure database access
    - Bind MySQL to localhost only, enforce strong authentication and validate
    user input to prevent SQL injection

5. Audit files for secrets
    - Avoid storing plaintext credentials in scripts or history files.

<!-- }}} -->

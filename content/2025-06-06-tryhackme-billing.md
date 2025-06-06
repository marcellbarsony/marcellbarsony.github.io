+++
title = "TryHackMe - Billing"
date = 2025-06-06

[taxonomies]
tags = ["tryhackme", "metasploit", "reverse shell", "suid", "privesc"]
+++

![TODO](/pictures/articles/thm/billing/00-cover.png)

**Billing** demonstrates how an unpatched vulnerability in MagnusBilling
can esaily be exxloited to gain remote command execution. This vulnerable machine
also highlights how a misconfigured instance of Fail2Ban can be used to gain
root access.


<!-- more -->


## Enumeration

<!-- Enumeration {{{-->

The nmap scan shows that the following ports are open on the target machine:

- 22 - OpenSSH
- 80 - Apache HTTP Server
- 3306 - MySQL (MariaDB)

![nmap](/pictures/articles/thm/billing/01-nmap.png)

The default script scan also reveals one disallowed entry in `robots.txt`.
When checking it out, a login page is being presented.

![website](/pictures/articles/thm/billing/02-website.png)

Poking around on the login page doesn't bring any success, but searching for the
software ([MagnusBilling](https://www.magnusbilling.org/)) being hosted
reveals that it has a [vulnerability](https://www.rapid7.com/db/modules/exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258/)
that could potentially be exploited to gain remote command execution.

<!-- }}} -->

# Exploitation

<!-- Exploitation {{{-->

The exploit for ([CVE-2023-30258](https://nvd.nist.gov/vuln/detail/CVE-2023-30258))
is present in the [Metasploit Framework](https://github.com/rapid7/metasploit-framework).

![site](/pictures/articles/thm/billing/03-msfconsole.png)

Searching for `magnus` gives a result as expected.

![msf-search](/pictures/articles/thm/billing/04-msf-search.png)

The local (LHOST) and the remote hosts (RHOST) should be configured to the
corresponding IPs of the attacking and the target machines.

![msf-conf](/pictures/articles/thm/billing/05-msf-conf.png)

Launching the exploit spawns a meterpreter shell on the system allowing commands
to be executed remotely.

![msf-run](/pictures/articles/thm/billing/06-msf-run.png)

The current session runs in the name of the user `asterisk` with the `uid` of
1001.

![msf-shell](/pictures/articles/thm/billing/07-msf-shell.png)

The user flag can be retrieved from `/home/magnus/user.txt`.

![user-flag](/pictures/articles/thm/billing/08-user-flag.png)

<!-- }}} -->

## Privilege escalation

<!-- Privilege escalation {{{-->

To spawn a proper reverse shell, a netcat listener should be set up to listen to
the incoming connection on an arbitrary port - `1234` in this case.

![nc-listen](/pictures/articles/thm/billing/09-nc-listen.png)

Connection can be initiated from the target machine back to the attacker.

![connect](/pictures/articles/thm/billing/10-connect.png)

Upon capturing the connection, the simple shell can be upgraded to an
interactive one.

Spawn a Bash shell using python:
```sh
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Set the terminal's type to `xterm` by setting the `TERM` environment variable:
```sh
export TERM=xterm
```

Suspend the current process with `Ctrl + Z`.

Modify the terminal's settings:
```sh
stty raw -echo; fg
```

- `raw`: Make the terminal work in raw mode to send everything directly to the
shell.
- `-echo`: Turn off echoing input characters, so typed characters aren't shown
on the screen.
- `fg`: Bring back the previously suspended process to the foreground.

![interactive-shell](/pictures/articles/thm/billing/11-interactive-shell.png)

Issuing `sudo -l` reveals that it is possible to execute
`/usr/bin/fail2ban-client` as the `root` user.

![sudo-l](/pictures/articles/thm/billing/12-sudo-l.png)

[Juggernaut-Sec's article](https://juggernaut-sec.com/fail2ban-lpe/) discloses
how Fail2Ban's configuration files could be used to elevate privileges on the
system.

As the first step, the aforementioned files should be copied to a temporary
directory.

![f2b-conf](/pictures/articles/thm/billing/13-f2b-conf.png)

Then, the custom configuration files need to be set up in a way, to execute
a custom action when triggered.

![f2b-exploit](/pictures/articles/thm/billing/14-f2b-exploit.png)

- Create a shell script in `/tmp/script`
- Copy `/bin/bash` to `/tmp/bash` and set the `setuid` bit

- Create a custom definition for Fail2Ban
- `actionstart` triggers `/tmp/script` upon a failed login attempt

- Add a custom jail configuration to Fail2Ban
- Tells Fail2Ban to use the custom action (`/tmp/script`)

- Create an empty filter for the custom jail to trigger the jail

Once the exploit has been set up, the Fail2Ban should be called to restart the
service with the custom configuration.

![f2b-restart](/pictures/articles/thm/billing/15-f2b-svc-restart.png)

After calling the service, the SUID binary created at `/tmp/bash` should be
executed with the `-p` flag.

![root](/pictures/articles/thm/billing/16-root.png)

The root flag can be read from `/root/root.txt`.

![root-flag](/pictures/articles/thm/billing/17-root-flag.png)

<!-- }}} -->

## Mitigation

<!-- Mitigation {{{-->

1. Update MagnusBilling
    - Apply security patches to the vulnerable software to address the RCE
    vulnerability

2. Limit Fail2Ban configuration
    - Ensure that Fail2Ban configuration files are not writeable by unauthorized
    users

3. Limit SUID binaries
    - Prevent unnecessary binaries from having the SUID bit set.
    - Implement file integrity monitoring to detect changes to SUID binaries

4. Limit Sudo permissions
    - Restrict the usage of `sudo` to the necessary commands only

<!-- }}} -->

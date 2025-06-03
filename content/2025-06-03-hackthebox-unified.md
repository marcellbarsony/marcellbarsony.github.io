+++
title = "HackTheBox - Unified"
date = 2025-06-03

[taxonomies]
tags = ["HackTheBox", "Log4j", "MongoDB", "reverse-shell", "privesc"]
+++

[![TODO](/pictures/articles/htb/unified/00-cover.png)](https://www.hackthebox.com/achievement/machine/447801/441)


Unified is introducing the exploitation of one of the biggest vulnerabilities of
2021, [Log4Shell](https://en.wikipedia.org/wiki/Log4Shell), also known as Log4j.
This box demonstrates how to exploit Log4j in the widely used UniFi network
monitoring system to gain reverse shell by manipulating a POST header.


<!-- more -->


## Enumeration

<!-- Enumeration {{{-->
The nmap scan shows that port `8080` is running a `http-open-proxy` that is
redirecting incoming requests to port `8443`.

![nmap](/pictures/articles/htb/unified/01-nmap.png)

Visiting webpage at `https://10.129.43.28:8443` displays the login page of the
**UniFi Network Application**, along with its version number `6.4.54`,
developed by [Ubiquiti](https://ui.com/download/unifi).

Googling its version number reveals that this specific version if vulnerable to
**Log4Shell** ([CVE-2021-44228](https://nvd.nist.gov/vuln/detail/CVE-2021-44228)),
also known as **Log4j**.

![website](/pictures/articles/htb/unified/02-website.png)

<!-- }}} -->

## Exploitation

<!-- Exploitation {{{-->

According to the article, the login POST request needs to be intercepted,
so that it can be modified and the malicious payload can be added.

![website](/pictures/articles/htb/unified/03-login-request.png)

The value of the `remember` parameter should be enclosed in quotation marks
(`"`) to ensure it is parsed as a string, rather than as a JSON object.

![payload](/pictures/articles/htb/unified/04-payload.png)

The response contains `api.errInvalidPayload` however, despite the error
message, the payload is successfully executed.

![response](/pictures/articles/htb/unified/05-response.png)

`tcpdump` confirms that the connection was indeed received on port `389` (LDAP).

![tcpdump](/pictures/articles/htb/unified/06-tcpdump.png)

Prior to crafting the payload, I have installed `Open-JDK` by issuing
`sudo apt install openjdk-11-jdk -y` and maven with
`sudo apt-get install maven`.

Then, I have cloned and built the package. The build process will create a file
called `RogueJndi-1.1.jar` in the `rogue-jndi/target` directory.
```sh
git clone https://github.com/veracode-research/rogue-jndi
cd rogue-jndi
mvn package
```

The payload will be responsible for creating a shell on the target system
and will be [Base64 encoded](https://en.wikipedia.org/wiki/Base64).

![payload-encode](/pictures/articles/htb/unified/07-payload-encode.png)

The payload should be then passed to Rogue-JNDI, along with a chosen port
(`1234`), and the IP address of the attacker machine (`--hostname`).

The server is now listening on port `1389` to the incoming connection.

![rogue-jndi](/pictures/articles/htb/unified/08-rogue-jndi.png)

To capture the reverse shell, a `netcat` listener should be opened listening on
port `1234`, as defined in the payload.

![netcat](/pictures/articles/htb/unified/09-netcat.png)

Now, the payload should be modified include port `1389`
in Burp Suite's Repeater.

![execute](/pictures/articles/htb/unified/10-execute.png)

Upon receiving the incoming connection, the `netcat` listener spawns
a reverse shell, which can be upgraded to an interactive shell with
`script /dev/null -c bash`.

![interactive-shell](/pictures/articles/htb/unified/11-interactive-shell.png)

The currently authenticated user is `unifi` with `uid=999`.

![user-enumeration](/pictures/articles/htb/unified/12-user-enumeration.png)

The user flag can be found in `/home/michael/user.txt`.

![user-flag](/pictures/articles/htb/unified/13-user-flag.png)

<!-- }}} -->

## Privilege escalation

<!-- Privilege escalation {{{-->

After a bit of searching on the internet, MongoDB is being used by UniFi to
store and share the SSH secrets between appliances. MongoDB is indeed running on
the system on port `27117`.

![mongo](/pictures/articles/htb/unified/14-mongo.png)

With the `mongo` command line utility the hash of the `administrator`'s password
can be revealed in the `x_shadow` variable.

![admin](/pictures/articles/htb/unified/15-admin.png)

This discovered hash can be replaced, but prior to that,
a new password (`pass1234`) must be created and hashed.

![pass-hash](/pictures/articles/htb/unified/16-pass-hash.png)

The current hash can be replaced using the following `mongo` command.

![update-pass-hash](/pictures/articles/htb/unified/17-update-pass-hash.png)

With the new password hash in place, it is now possible to authenticate to the
UniFi Network Application.

![login](/pictures/articles/htb/unified/18-login.png)

The authentication is successful, and the dashboard is displayed.

![dashboard](/pictures/articles/htb/unified/19-dashboard.png)

Under settings, the password for the SSH connection can be read in plain text.

![passwd](/pictures/articles/htb/unified/20-passwd.png)

With the found password it is now possible to authenticate to the target
via SSH.

![ssh](/pictures/articles/htb/unified/21-ssh.png)

The `root` flag can be found in the home directory of the `root` user.

![root-flag](/pictures/articles/htb/unified/22-root-flag.png)

With the `root` flag, the machine is now [pwned](https://www.hackthebox.com/achievement/machine/447801/441).

<!-- }}} -->

## Mitigation

<!-- Mitigation {{{-->
1. Update the Log4j library
    - Upgrade to the latest possible version of Log4j, where the vulnerability
    has been fixed

2. Network segmentation
    - Limit the exposure of critical services to trusted networks (e.g., VPN)
    only

3. Web Application Firewall
    - Implement a WAF to block malicious payloads attempting JNDI injection

4. Secure MongoDB access
    - Require authentication for MongoDB and restrict remote access

5. Limit privileges
    - Run services with the least privilege necessary
<!-- }}} -->

+++
title = "Hack The Box - Archetype"
date = 2025-05-24

[taxonomies]
tags = ["hackthebox", "sql", "smb", "netcat", "reverse-shell", "privesc"]
+++

![archetype](/pictures/articles/htb/archetype/cover.png)

Archetype is a Tier II Windows-based machine from Hack The Box's
[Starting Point module](https://app.hackthebox.com/starting-point)
designed to teach key penetration testing techniques such as
leveraging SMB to gain access, spawning a reverse shell and escalating
privileges to complete the machine.


<!-- more -->


## Enumeration

<!-- Enumeration {{{-->

Scanning **Archetype** [10.129.213.185] with [nmap](https://nmap.org/) reveals
that the target ([Windows Server 2019](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019))
is running an [SQL server](https://www.microsoft.com/en-us/sql-server/sql-server-2017)
on port 1433.

![nmap-scan](/pictures/articles/htb/archetype/enum-01.png)

- `-sC`: Script scan
- `-sV`: Version detection

The open [SMB](https://en.wikipedia.org/wiki/Server_Message_Block) port can be
enumerated further with [smbclient](https://www.samba.org/samba/docs/current/man-html/smbclient.1),
which locates the shares `ADMIN$` and `C$`. Unfortunately, they cannot be
accessed as they require elevated privileges however, there is an additional
share named `backups`, which is publicly accessible.

![smbclient](/pictures/articles/htb/archetype/enum-02.png)

- `-N`: Suppress password prompt
- `-L`: List available services
- Additional backslashes are required to escape Windows's UNC path

retrieved with the `get prod.dtsConfig` command.
`backups` contains a configuration file named `prod.dtsConfig` that can be

![smbclient](/pictures/articles/htb/archetype/enum-03.png)

The contents of the file discloses the user `sql_svc` and the
corresponding unencrypted (plain-text) password `M3g4c0rp123`.

![smbclient](/pictures/articles/htb/archetype/enum-04.png)

<!-- }}} -->

## Foothold

<!-- Foothold {{{-->

The discovered credentials can be used to connect and authenticate to the MSSQL
server using the script `mssqlclient.py` from the [Impacket](https://github.com/fortra/impacket)
collection.

The command `SELECT is_srvrolemember('sysadmin');` indicates the assigned role
on the server: `1` (`True`) means that the current login has `sysadmin` role.

![smbclient](/pictures/articles/htb/archetype/foothold-01.png)

`xp_cmdshell` allows the execution of Windows command shell commands directly
from the SQL Server environment. 

Issuing the command `EXEC xp_cmdshell 'net user';` helps determine whether
`xp_cmdshell` is enabled, as this feature is disabled by default
for security reasons.

![smbclient](/pictures/articles/htb/archetype/foothold-02.png)

`xp_cmdshell` can be activated with the following set of commands.

```sh
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
sp_configure;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

It is now possible to execute commands on the target machine.

![smbclient](/pictures/articles/htb/archetype/foothold-03.png)

<!-- }}} -->

## Reverse shell

<!-- Reverse shell {{{-->
After gaining command execution, a reverse shell can be spawned using [Netcat](https://github.com/int0x33/nc.exe).

To upload `nc64.exe` to the target, the file must be served from the attacker
machine via a simple HTTP server.

![http-server](/pictures/articles/htb/archetype/reverse-shell-01.png)

The file can be retrieved from the attacker's machine by issuing
the following PowerShell command via `xp_cmdshell`.

![get-nc64](/pictures/articles/htb/archetype/reverse-shell-02.png)

The Netcat listener should be started to receive the incoming connection.

![reverse-shell](/pictures/articles/htb/archetype/reverse-shell-03.png)

- `l`: Listen mode
- `v`: Verbose mode
- `n`: No DNS resolution
- `p`: Port number

After uploading, `nc64.exe` can be executed so that it binds to
the Netcat listener.

![reverse-shell](/pictures/articles/htb/archetype/reverse-shell-04.png)

Netcat has received the incoming connection and spawned
an interactive reverse shell.

![reverse-shell](/pictures/articles/htb/archetype/reverse-shell-05.png)

The user flag can be found in the user's Desktop directory.

![user-flag](/pictures/articles/htb/archetype/user-flag-01.png)

<!-- }}} -->

## Privilege escalation

<!-- Privilege escalation {{{-->

Privilege escalation is required to gain a higher level of control over the
system. Since the currently logged-in user account is also a service account,
it may be worth checking for its previously executed commands stored in the
PowerShell history file.

![privesc-01](/pictures/articles/htb/archetype/privesc-01.png)

With the obtained administrator password, it is now possible to connect to the
target machine using [PsExec](https://learn.microsoft.com/en-us/sysinternals/downloads/psexec)
and obtain the root flag from the administrator account's Desktop directory.

![root-flag](/pictures/articles/htb/archetype/root-flag-01.png)

<!-- }}} -->

## Mitigation

<!-- Mitigation {{{-->

1. Disable SMB shares
    - Restrict SMB shares such as `ADMIN$` and `C$`

2. Disable `xp_cmdshell`
    - Use `sp_configure` to make sure it remains disabled

3. Enforce strong password
    - Ensure service accounts (e.g., `sql_svc`) have a strong password
    - Avoid storing credentials in clear text

4. Implement least privilege
    - Restrict user and service account privileges

<!-- }}} -->

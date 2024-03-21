+++
title = "Metasploitable - File upload vulnerability"
date = 2019-02-02

[taxonomies]
tags = ["metasploitable", "showcase"]
+++

## Notes

- Simple vulnerability

## Lab setup

The virtual machines are placed onto the same isolated internal network where they can only communicate with each other.<br>
On the isolated internal network, the VirtualBox DHCP server assigns IP addresses to the machines:

```sh
marci@arch$ vboxmanage dhcpserver add --network=intnet --server-ip=10.38.1.1 --lower-ip=10.38.1.110 --upper-ip=10.38.1.120 --netmask=255.255.255.0 --enable
```

The **Kali** machine received the IP address `10.38.1.110`.

Launching **Mr. Robot**, we're presented with a login screen that cannot be bypassed as the login credentials aren't known.

![mr_robot](https://www.dropbox.com/s/rftjad3vikt9yyp/mr_robot.jpg?dl=1)


### Nmap

**Nmap** can be used to discover the vulnerable machine on the network:

```sh
kali@kali$ nmap -sS -T4 10.38.1.110-120
```

```sh
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-07 15:04 EDT
Nmap scan report for 10.38.1.110
Host is up (0.0000060s latency).
All 1000 scanned ports on 10.38.1.110 are in ignored states.
Not shown: 1000 closed tcp ports (reset)

Nmap scan report for 10.38.1.111
Host is up (0.00050s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE  SERVICE
22/tcp  closed ssh
80/tcp  open   http
443/tcp open   https
MAC Address: 08:00:27:FE:07:51 (Oracle VirtualBox virtual NIC)

Nmap done: 11 IP addresses (2 hosts up) scanned in 32.14 seconds
```

**Nmap** found the vulnerable machine and reported it with the IP address `10.38.1.111`.<br>
Port [80](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=80) and [443](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=443) are well-known ports used to establish connection with **web servers**.<br>
Looking up `10.38.1.111` in a web browser, we're presented with the clone of the promotion website of the [Mr. Robot TV series](https://www.imdb.com/title/tt4158110/).

![fsociety](https://www.dropbox.com/s/mh5v8lc88k9r8tx/fsociety.png?dl=1)

Checking the displayed commands won't get us anywhere close to the first flag.

## Enumeration

```sh
nmap


```

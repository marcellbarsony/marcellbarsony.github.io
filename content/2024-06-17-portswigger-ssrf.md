+++
title = "Server-side Request Forgery [SSRF]"
date = 2024-06-17

[taxonomies]
tags = ["webapp", "portswigger", "burp-suite", "server-side", "ssrf" ]
+++


![ssrf](/pictures/articles/server-side-request-forgery/server-side-request-forgery.svg)

**Server-side request forgery (SSRF)** allows an attacker to cause the
server-side application to make a HTTP requests to an unintended, arbitrary
internal or external) location.


<!-- more -->


## Exploitation



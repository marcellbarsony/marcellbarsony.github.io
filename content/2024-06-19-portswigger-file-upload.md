+++
title = "File Upload vulnerabilities"
date = 2024-06-17

[taxonomies]
tags = ["webapp", "portswigger", "burp-suite", "server-side", "file upload" ]
+++


![file-upload](/pictures/articles/file-upload/file-upload.png)

**File Upload vulnerabilities** occur when an attacker can upload arbitrary
malicious files to a web server without proper validation.
This includes insufficient checks on the file's contents, type or size.
Once uploaded, these files may be executed, potentially causing havoc on the
web server and on its underlying infrastructure.


<!-- more -->


The impact of this vulnerability largely depends on the following two key
factors:
- Which aspects of the file are not properly validated
  (e.g., contents, type, size, etc.)
- The restrictions applied after the file has been uploaded

## Exploitation

<!-- LAB 1 {{{-->
### [LAB 1 - Remote code execution via web shell upload](https://portswigger.net/web-security/learning-paths/file-upload-vulnerabilities/exploiting-unrestricted-file-uploads-to-deploy-a-web-shell/file-upload/lab-file-upload-remote-code-execution-via-web-shell-upload)

With the credentials `wiener:peter` we can log in to our own account
and on our profile there is a feature to upload an avatar.

![file-upload](/pictures/articles/file-upload/lab1-1.png)

Once the arbitrary avatar is uploaded, it is possible to intercept the
requests our browser sends to the server to fetch the picture.

Intercepting these HTTP `GET` request, it's possible to filter them down
to Image MIME types.

![file-upload](/pictures/articles/file-upload/lab1-2.png)

Due to the absence of proper validation, arbitrary malicious files
can be uploaded using the same functionality.

The following PHP one-liner should retrieve the target's secret from their home
directory:
`<?php echo file_get_contents('/home/carlos/secret'); ?>`

By replacing the filename of the avatar from the previous GET request,
the content of the script can now be invoked.

![file-upload](/pictures/articles/file-upload/lab1-3.png)

The server runs the script and sends a response with carlos's credentials:
`sPOb66xdZJHNIi3hCreeg2QMM0eXGic2`.

![file-upload](/pictures/articles/file-upload/lab1-4.png)

<!-- }}} -->

<!-- LAB 2 {{{-->
<!--### [LAB 2 - ]()-->


![file-upload](/pictures/articles/file-upload/lab1-1.png)
<!-- }}} -->

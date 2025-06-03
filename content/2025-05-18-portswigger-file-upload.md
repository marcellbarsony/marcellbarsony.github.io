+++
title = "Portswigger - File Upload vulnerabilities"
date = 2025-05-18

[taxonomies]
tags = ["webapp", "portswigger", "burp suite", "server-side", "file upload"]
+++

![file-upload](/pictures/articles/portswigger/file-upload/file-upload.png)

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

![file-upload](/pictures/articles/portswigger/file-upload/lab-1-1.png)

Once the arbitrary avatar is uploaded, it is possible to intercept the
requests our browser sends to the server to fetch the picture.

Intercepting these HTTP `GET` requests, it's possible to filter them down
to Image MIME types.

![file-upload](/pictures/articles/portswigger/file-upload/lab-1-2.png)

After pinpointing out the responsible request, it could be sent to the Repeater
so that it can be modified to suit our needs later on.

![file-upload](/pictures/articles/portswigger/file-upload/lab-1-3.png)

Due to the absence of proper validation, arbitrary malicious files
can also be uploaded using the same website functionality.

Instead of a picture, the following PHP one-liner can also be uploaded
that will retrieve the target's secret from their home directory:

![file-upload](/pictures/articles/portswigger/file-upload/lab-1-4.png)

By replacing the filename of the avatar from the previously captured
`GET` request, the content of the malicious script can now be invoked.

![file-upload](/pictures/articles/portswigger/file-upload/lab-1-5.png)

The server runs the exploit and sends a response with `carlos`'s credentials:
`sPOb66xdZJHNIi3hCreeg2QMM0eXGic2`.

![file-upload](/pictures/articles/portswigger/file-upload/lab-1-6.png)
<!-- }}} -->

<!-- LAB 2 {{{-->
<!-- ### [LAB 2 - Web shell upload via Content-Type restriction bypass](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/file-upload-apprentice/file-upload/lab-file-upload-web-shell-upload-via-content-type-restriction-bypass) -->


<!-- }}} -->

<!-- Mitigation {{{-->
## Mitigation

1. Input Validation & Restriction
    - Allow only specific file types and extensions based on a strict allowlist
    - Validate file MIME types and magic bytes
      instead of relying solely on extensions
    - Reject files with double extensions (e.g., file.php.jpg).

2. Storage & Execution Restrictions
    - Never store uploaded files in a web-accessible directory
    (e.g., inside public_html)
    - Store files outside the web root or use a cloud storage service
    - Disable execution permissions on upload directories
      to prevent running malicious scripts

3. File Size & Content Limitations
    - Set maximum file size limits to prevent denial-of-service attacks
    - Restrict file uploads by user role or authentication level

4. Secure File Naming & Path Handling
    - Generate random file names instead of using user-supplied names
      to prevent overwriting files
    - Remove or sanitize metadata from uploaded files
      to prevent information leaks
    - Prevent directory traversal attacks by sanitizing file paths

5. Content-Type & Headers Validation
    - Validate `Content-Type` headers and enforce strict response headers
    - Use Content-Disposition: attachment to prevent files
      from executing in the browser

6. Implement Strong Authentication & Authorization
    - Require authentication before allowing uploads
    - Implement role-based access control (RBAC) to restrict upload permissions

7. Log & Monitor File Uploads
    - Log all upload attempts, including IP addresses, file names, and users
    - Implement alerts for suspicious file types or excessive upload attempts
<!-- }}} -->

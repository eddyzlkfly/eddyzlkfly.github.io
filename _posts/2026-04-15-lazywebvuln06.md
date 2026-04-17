---
title: "LazyWeb – VULN 6: Insecure File Upload"
date: 2026-04-14
categories: [Writeups, WebPentest, Web Exploitation]
tags: [web security, pentesting, owasp, burp suite, vulnerability]
description: Analysis of an insecure file upload vulnerability in LazyWeb, allowing arbitrary file uploads and potential security risks.
image: /assets/img/lazyweb-cover.png
---
## Description

The web application allows users to upload profile avatars through the **Update Profile** functionality. However, the application does not properly validate the content of uploaded files.

Although the server renames uploaded files to a `.png` extension, it does not verify whether the uploaded file is actually a valid image. This allows attackers to upload arbitrary files disguised as image files.

As a result, malicious files can be stored on the server even though they may not be executed directly.

## Discovery Process

### Step 1 – Identify the File Upload Feature

The application provides an **Update Profile** page where users can upload an avatar image.

The uploaded file is submitted through the following form field:

![/assets/img/imgvuln6.png](/assets/img/imgvuln6.png)

### Step 2 — Test File Upload Validation

To test whether the server properly validates uploaded files, a malicious PHP file disguised as an image was created.

Uploaded file name:

`shell.php.png`

File content:

```php
&lt;?php system($_GET['cmd']); ?&gt;
```

The file was uploaded successfully through the avatar upload functionality.

### Step 3 — Verify Uploaded File Location

Analysis of the application source code revealed that uploaded avatar files are stored in the following directory:
```
/user/avatar/
```
The uploaded file is automatically renamed based on the user ID.

Example file location:
```
http://localhost/user/avatar/1.png
```
## Proof of Concept (PoC)

After uploading the malicious file, the file was accessible through the web server.

When accessed in the browser, the image failed to render because the uploaded file does not contain valid image data.

Example browser message:

![/assets/img/imgvuln6(1).png](/assets/img/imgvuln6(1).png)

This confirms that arbitrary files can be uploaded to the server without proper validation.

## Impact

Successful exploitation of this vulnerability allows an attacker to:

- Upload arbitrary files to the server
- Store malicious content on the web server
- Potentially overwrite existing files
- Use the uploaded files in combination with other vulnerabilities such as **Local File Inclusion (LFI)**

Although the uploaded file is not executed as PHP due to the enforced `.png` extension, the ability to upload arbitrary files still represents a security risk.

## Severity

<span style="color: #f8cb17;">**Medium**</span>

The vulnerability allows attackers to upload arbitrary files but does not directly lead to code execution due to file extension restrictions.
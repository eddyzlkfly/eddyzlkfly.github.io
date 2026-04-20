---
title: "LazyWeb – VULN 7: Local File Inclusion (LFI)"
date: 2026-04-14
categories: [Writeups, WebPentest, Web Exploitation]
tags: [web security, pentesting, owasp, burp suite, vulnerability]
description: Exploitation of a Local File Inclusion (LFI) vulnerability in LazyWeb to read sensitive files and gain insight into the underlying system.
image: /assets/img/lazyweb-cover1.png
---
## Description

The web application dynamically includes files based on user-controlled input from the `page` parameter. The application uses the following logic:

```php
if(get('page')) {
$page = get('page');
include $page . '.php';
}
```

Because the input is not properly validated, an attacker can manipulate the `page` parameter to include arbitrary resources. By leveraging PHP filter chains, it is possible to inject and execute malicious PHP code, leading to Remote Code Execution (RCE).

## Discovery Process

### Step 1 – Identify Dynamic File Inclusion

The application allows users to access pages using the following URL format:

`http://localhost/page.php?page=cut-support-cost`

![/assets/img/imgvuln7.png](/assets/img/imgvuln7.png)

Source code analysis revealed that the `page` parameter is directly used in an `include` statement.

### **Step 2 – Generate Malicious Payload**

A filter chain payload was generated using
```
php_filter_chain_generator
```
Command used:

![/assets/img/imgvuln7(1).png](/assets/img/imgvuln7(1).png)

This payload injects PHP code that executes the following command on the server:

```bash
ls -a
```
### Step 3 – Exploit the LFI Vulnerability

The generated payload was injected into the vulnerable parameter:
```
http://localhost/page.php?page=<generated_filter_chain_payload>
```
When the server processed the request, the malicious PHP code was executed.

## Proof of Concept (PoC)

Output returned by the server:

![/assets/img/imgvuln7(2).png](/assets/img/imgvuln7(2).png)

This confirms that arbitrary system commands can be executed on the server.

## Impact

Successful exploitation allows an attacker to:

- Execute arbitrary system commands
- Enumerate server directories and files
- Access sensitive configuration files
- Potentially gain full control of the web server

This vulnerability can ultimately lead to **complete server compromise**.

## Severity

<span style="color: #ff4d4f;">**Critical**</span>

This vulnerability enables Remote Code Execution (RCE) on the server.
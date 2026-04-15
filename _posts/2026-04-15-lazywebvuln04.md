---
title: "LazyWeb – VULN 4: Command Injection → Remote Code Execution (RCE)"
date: 2026-04-14
categories: [Writeups, WebPentest]
tags: [web security, pentesting, owasp, burp suite, vulnerability]
description: Analysis of a command injection vulnerability in LazyWeb, leading to arbitrary command execution and remote code execution (RCE).
image: /assets/img/lazyweb-cover.png
---
## Description

The web application contains a command injection vulnerability in the **Add Sites** functionality. The application allows users to submit a URL which is later used by the server to execute a system command to check the availability of the website.

The user-controlled input is not properly validated or sanitized before being used in a system command. As a result, an attacker can inject arbitrary commands that will be executed on the server.

This vulnerability allows attackers to perform **Remote Code Execution (RCE)** on the server.

## Discovery Process

### Step 1 – Identify the Add Sites Functionality

During testing, the **Add Sites** feature was analyzed. This feature allows users to submit a website URL which is then stored and can be interacted with using the **Ping** function.

The application appears to execute a system command to ping the specified URL.

![/assets/img/imgvuln4.png](/assets/img/imgvuln4.png)

### Step 2 – Test for Command Injection

To test for command injection, a malicious payload was inserted into the URL field.

Payload used:
```bash
localhost ; ls -a
```
The semicolon (`;`) allows execution of an additional command after the original command.

### Step 3 – Trigger Command Execution

After adding the malicious site, the **Ping** button was clicked to trigger the command execution.

The application executed a command similar to the following:
```bash
ping -c 1 localhost ; ls -a
```
Because the user input was not sanitized, the injected command was executed by the server.

## Proof of Concept (Poc)

Server response:

![/assets/img/imgvuln4(1).png](/assets/img/imgvuln4(1).png)

The server returned the list of files from the web application directory, confirming that the injected command was successfully executed.

## Impact

Successful exploitation of this vulnerability allows an attacker to:

- Execute arbitrary system commands on the server
- Enumerate server directories and files
- Access sensitive configuration files
- Potentially gain full control of the web server

This vulnerability can ultimately lead to **complete server compromise**.

## Severity

<span style="color: #ff4d4f;">**Critical**</span>

Command injection vulnerabilities allow attackers to execute arbitrary commands on the server, which may lead to full system compromise.
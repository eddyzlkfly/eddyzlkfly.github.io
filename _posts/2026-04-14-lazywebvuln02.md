---
title: "LazyWeb – VULN 2: Hardcoded Database Credentials Leading to Database Access"
date: 2026-04-14
categories: [Writeups, WebPentest, Web Exploitation]
tags: [web security, pentesting, owasp, burp suite, vulnerability]
description: Discovery of hardcoded database credentials in LazyWeb, enabling direct database access and potential compromise of sensitive data.
image: /assets/img/lazyweb-cover1.png
---
## Description

During the penetration testing process, database credentials were discovered in the application source code. Combined with an exposed MySQL service, these credentials allowed direct access to the backend database.

## Discovery Process

Step 1 — Network Scanning

Network scanning was performed using Nmap to identify open ports and running services.

![/assets/img/imgvuln2(2).png](/assets/img/imgvuln2(2).png)

This indicates that the MySQL service is accessible.

Step 2 — Source Code Analysis

After retrieving the source code through the exposed `.git` repository, a search was performed to locate database configuration values.

Command & Results:

![/assets/img/imgvuln2(3).png](/assets/img/imgvuln2(3).png)

Step 3 — Database Access

Using the discovered credentials, a connection to the database server was established.

![/assets/img/imgvuln2.png](/assets/img/imgvuln2.png)

Successful authentication granted access to the database shell.

Step 4 — Database Enumeration

Once connected, database enumeration was performed.

![/assets/img/imgvuln2(1).png](/assets/img/imgvuln2(1).png)

This confirmed access to the application database.

## Impact

An attacker who obtains the exposed credentials could directly connect to the database server and access sensitive information stored within the application.

Potential impacts include:

- Disclosure of user information
- Retrieval of password hashes
- Modification or deletion of data

## Recommendation

- Avoid storing credentials in source code repositories
- Use environment variables or secure secret management
- Restrict database access to internal networks only
- Implement proper access control policies

## Severity

<span style="color: #ff4d4f;">**Critical**</span>
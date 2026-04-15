---
title: "LazyWeb – VULN 10: Time-Based Blind SQL Injection in Registration Function"
date: 2026-04-14
categories: [Writeups, WebPentest]
tags: [web security, pentesting, owasp, burp suite, vulnerability]
description: Exploitation of a time-based blind SQL injection vulnerability in LazyWeb to extract sensitive data from the database through time-delay techniques.
image: /assets/img/lazyweb-cover.png
--- 
## Description

The registration functionality does not properly sanitize user input before

including it in SQL queries. By injecting a time-based SQL payload using the

`SLEEP()` function, the server response was noticeably delayed.

This confirms that user-controlled input is directly processed by the

database engine, allowing attackers to manipulate SQL queries.

## Proof of Concept (PoC)

Normal request:

```sql
usermail=test@test.com
```

SQL injection payload:

```sql
[usermail=test@test.com](mailto:usermail=test@test.com)' AND SLEEP(510)-- -
```

Observed behaviour:

Normal response: immediate

![/assets/img/imgvuln10.png](/assets/img/imgvuln10.png)

Injected payload: delayed response (~10 seconds)

![/assets/img/imgvuln10(1).png](/assets/img/imgvuln10(1).png)

This indicates a **time-based blind SQL injection vulnerability**.

## Impact

- Attackers can manipulate backend SQL queries through unsanitized user input.
- Sensitive information stored in the database may be exposed, including user emails and password hashes.
- Attackers may be able to dump the entire database using automated tools such as sqlmap.
- User credentials could potentially be extracted and used to gain unauthorized access to user accounts.
- The vulnerability may allow attackers to bypass authentication mechanisms.
- In severe cases, attackers may escalate privileges and gain administrative access to the application.
- Overall, the confidentiality and integrity of the application's data may be compromised.

## Severity

<span style="color: #ff922b;">**High**</span>
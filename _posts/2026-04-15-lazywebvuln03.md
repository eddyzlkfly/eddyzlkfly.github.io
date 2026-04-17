---
title: "LazyWeb – VULN 3: Insecure Direct Object Reference (IDOR) via Cookie Manipulation"
date: 2026-04-14
categories: [Writeups, WebPentest, Web Exploitation]
tags: [web security, pentesting, owasp, burp suite, vulnerability]
description: Exploitation of an IDOR vulnerability in LazyWeb by manipulating cookies, allowing access to other users’ data without proper authorization.
image: /assets/img/lazyweb-cover.png
---
## **Description**

The application relies on a client-controlled cookie (`user_id`) to determine the identity of the logged-in user. Since this value can be modified by the client, an attacker can manipulate the cookie to access other user accounts.

## Discovery Process

Step 1 — Source Code Analysis

After obtaining the application source code from the exposed Git repository of **LazyWeb**, the code was analyzed to identify potential security weaknesses.

During the analysis, the following database query was found:
```sql
SELECT * FROM tbl_users WHERE id = cookie('user_id')
```
This indicates that the application determines the logged-in user based solely on the value of the `user_id` cookie.

Step 2 — Identifying Potential Vulnerability

Since cookies are stored on the client side, they can be modified by the user. If the application does not validate the cookie value properly, an attacker may be able to impersonate another user.

Step 3 — Cookie Manipulation

The application was accessed through a browser, and cookies were inspected using the browser developer tools.

Original cookie:

`user_id=2`

The cookie value was modified to:

`user_id=1`

Step 4 — Verification

After refreshing the page, the application loaded a different user account, confirming that the application trusts the client-controlled cookie without proper authorization checks.

`user: ez`

![/assets/img/imgvuln3(1).png](/assets/img/imgvuln3(1).png)

`user: a`

![/assets/img/imgvuln3.png](/assets/img/imgvuln3.png)

## Impact

An attacker could modify the `user_id` cookie to access other user accounts, potentially leading to:

- Unauthorized access to sensitive information
- Account takeover
- Unauthorized modification of user data

## Recommendation

- Do not rely on client-controlled cookies for authentication.
- Use secure session management.
- Validate user identity on the server side.

## Severity

<span style="color: #ff922b;">**High**</span>
---
title: "LazyWeb – VULN 8: Cross-Site Request Forgery (CSRF) → XSS"
date: 2026-04-14
categories: [Writeups, WebPentest, Web Exploitation]
tags: [web security, pentesting, owasp, burp suite, vulnerability]
description: Identification and exploitation of a CSRF vulnerability in LazyWeb, enabling attackers to inject and execute malicious scripts (XSS) in the victim’s browser.
image: /assets/img/lazyweb-cover1.png
--- 
## Description

The application is vulnerable to **Cross-Site Request Forgery**, where an attacker can trick an authenticated user into performing unintended actions without their consent.

The vulnerability occurs because the application **does not implement any CSRF protection mechanisms**, such as:

- CSRF tokens
- Origin validation
- SameSite cookie restrictions

As a result, an attacker can craft a malicious web page that automatically submits a request to the vulnerable endpoint using the victim's authenticated session.

Since the victim's browser automatically includes session cookies with the request, the server processes the request as if it was initiated by the legitimate user.

## Discovery Process

### Step 1 — Accessing the Inbox Feature

The tester logged into the web application and navigated to the **Inbox** page to analyze how messages are sent between users.

`http://localhost/user/inbox.php`

This page allows authenticated users to send messages to other users.

### Step 2 — Intercepting HTTP Requests

To understand how the messaging functionality works, we need intercepted HTTP traffic using **Burp Suite**.

When a message was sent through the application, the following request was captured:

![/assets/img/imgvuln8.png](/assets/img/imgvuln8.png)

This request sends a message to another user.

### Step 3 — Analyzing the Request

The intercepted request was analyzed to identify whether any CSRF protection mechanisms were implemented.

The analysis revealed that the request **did not include any CSRF token** or other anti-CSRF validation mechanisms.

The application only relied on the session cookie:

```
PHPSESSID=…
```

for authentication.

### Step 4 — Crafting a CSRF Attack Page

To test whether the application was vulnerable, a malicious HTML page was created to replicate the request.

The following HTML code automatically sends a forged request:

![/assets/img/imgvuln8(1).png](/assets/img/imgvuln8(1).png)

### Step 5 — Hosting the Malicious Page

The malicious page was hosted using a simple HTTP server:

![/assets/img/imgvuln8(2).png](/assets/img/imgvuln8(2).png)

The attack page was then accessed via:
```
http://127.0.0.1:9000/csrf.html
```
### Step 6 — Triggering the Attack

When the victim visited the malicious page while logged into the application, the browser automatically submitted the forged request.

Since the browser included the victim's session cookie, the server processed the request as a legitimate action.

### Step 7 — Successful Exploitation

The forged request successfully created a message in the inbox.

The injected payload:

```jsx
<script>alert(1)</script>
```

executed when the message was viewed, confirming that the CSRF attack was successful and could also lead to **Stored XSS**.

## Proof of Concept (PoC)

The attacker-hosted malicious page was accessed while the victim was authenticated in the application.

The CSRF payload automatically submitted a forged request to the vulnerable endpoint.

As a result, a new message with the subject **“CSRF Attack”** was successfully created in the inbox without the victim’s interaction.

![/assets/img/imgvuln8(3).png](/assets/img/imgvuln8(3).png)

The injected payload was stored in the message field.

When the victim opened the message, the JavaScript payload executed automatically in the browser, triggering a popup alert. This confirms that the application is vulnerable to **Stored Cross-Site Scripting (XSS)** triggered through a CSRF attack.

![/assets/img/imgvuln8(4).png](/assets/img/imgvuln8(4).png)

## Impact

The vulnerability allows attackers to force authenticated users to perform unintended actions through a forged request. By exploiting this vulnerability, an attacker can send malicious messages on behalf of the victim without their consent.

Furthermore, the attacker can inject malicious JavaScript payloads which are stored in the application and executed when the message is viewed. This may allow attackers to:

- Execute arbitrary JavaScript in the victim's browser
- Steal session cookies
- Perform session hijacking
- Impersonate legitimate users
- Conduct further attacks against other users

This significantly compromises the confidentiality and integrity of user accounts within the application.

## Severity

<span style="color: #ff922b;">**High**</span>
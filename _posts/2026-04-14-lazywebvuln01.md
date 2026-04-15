---
title: "LazyWeb – VULN 1: Exposed Git Repository"
date: 2026-04-14
categories: [Writeups, WebPentest]
tags: [web security, pentesting, owasp, burp suite, vulnerability]
description: Identification and exploitation of an exposed Git repository in LazyWeb, leading to source code leakage and potential sensitive information disclosure.
image: /assets/img/lazyweb-cover.png
---
## Description
The `.git` directory was accessible through the web server. This allows attackers to retrieve repository metadata and potentially reconstruct the entire source code.

## Discovery Process

Directory enumeration was performed using:

![/assets/img/imgvuln1(2).png](/assets/img/imgvuln1(2).png)

![/assets/img/imgvuln1(1).png](/assets/img/imgvuln1(1).png)

The scan revealed an accessible `.git` directory.

Accessible paths:

- http://localhost/.git/HEAD
- http://localhost/.git/config

## Proof of Concept

Accessing the `.git/HEAD` file revealed the current branch reference:

![/assets/img/imgvuln1.png](/assets/img/imgvuln1.png)

The repository logs also revealed the original GitHub repository.

## Impact

Attackers can retrieve source code which may expose sensitive information such as credentials, API keys, or internal logic.

## Recommendation

- Remove `.git` directories from the web root.
- Ensure sensitive files are not publicly accessible.
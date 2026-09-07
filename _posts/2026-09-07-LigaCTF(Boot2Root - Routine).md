---
title: "OWASP Liga CTF 2026 - Routine (Boot2Root)"
date: 2026-05-29
categories: [CTF, Writeups, B2R]
tags: [OWASPKL]
description: A challenge featuring Grafana enumeration and exploitation of CVE-2021-43798, a path traversal vulnerability that allows unauthenticated arbitrary file reads. The attack chain leads to credential extraction from the Grafana database, SSH access, and privilege escalation through a root-executed cron job to obtain full system access.
image: /assets/img/OWASPKcover.png
---

## Network Reconnaissance

```bash
nmap -sV -sC 192.168.100.118 --vv
```

Resutls:

![/assets/img/imgLIGA.png](/assets/img/imgLIGA.png)

- `Port **22**` → SSH (OpenSSH 10.2p1)
- `Port **3000**` → HTTP (Grafana)

Only two ports were open. The primary target was the Grafana service running on port 3000.

## Enumeration

Identify Grafana Version & Directory Bruteforce

![/assets/img/imgLIGA(0).png](/assets/img/imgLIGA(0).png)

The output revealed **Grafana 8.3.0**, which is important because this version contains a critical vulnerability.

Discovered endpoints:

- `/login` → Grafana login page
- `/signup` → Registration page
- `/api` and `/apis` → API endpoints (401 Unauthorized)
- `/robots.txt` → Standard robots file

## Exploitation (CVE-2021-43798)

Grafana versions **8.x prior to 8.3.1** are vulnerable to a **Path Traversal** flaw within the `/public/plugins/` endpoint. This vulnerability allows to read arbitrary files outside the intended directory without authentication.

### Verify the Vulnerability Using `/etc/passwd`

Using Burp Suite Repeater:

![/assets/img/imgLIGA(1).png](/assets/img/imgLIGA(1).png)

> The application uses a MySQL plugin, so the plugin name `mysql` was used in the path.
> 

The server returned the contents of `/etc/passwd`, confirming the vulnerability.

### Download the Grafana Database

![/assets/img/imgLIGA(2).png](/assets/img/imgLIGA(2).png)

The `--path-as-is` option ensures that the `../` sequences are not normalized by curl.

### Extract Credentials

A suspicious table named **credentials** was discovered during the listing of available tables above.

Query the table:

![/assets/img/imgLIGA(3).png](/assets/img/imgLIGA(3).png)

The table contained plaintext credentials stored in the Grafana database.

## User Flag (SSH Access)

Attempt SSH login using the discovered credentials:

![/assets/img/imgLIGA(4).png](/assets/img/imgLIGA(4).png)

Login was successful.

Retrieve the user flag:

```bash
cat local.txt
```

## 🚩 User Flag

`OWASPKL{496d5373e7501c9aab3b2658bbad4c02}` 

## Privilege Escalation (Cron Job Hijacking)

After multiple enumeration, I check the cron jobs:

![/assets/img/imgLIGA(5).png](/assets/img/imgLIGA(5).png)

![/assets/img/imgLIGA(6).png](/assets/img/imgLIGA(6).png)

The script `/opt/backup.sh` is executed by root every minute.

## Vulnerability Identified

The root-owned backup script executes a Python file located inside the user's home directory:

`/home/tellytubby/Downloads/userbackup.py`

Since the file is owned by the user, it can be modified and will later be executed by root through the cron job.

## Exploitation

Replace the original script with a malicious payload:

![/assets/img/imgLIGA(7).png](/assets/img/imgLIGA(7).png)

Wait for the cron job to execute.

Verify that the SUID bit has been applied:

```bash
-rwsr-sr-x 1 root root ... /bin/bash
```

Launch a privileged shell and the whoami result are `root` .

### Why use the `p` flag?

The `-p` option tells Bash to preserve the effective privileges of the file owner (root) instead of dropping them to the current user.

Retrieve the root flag:

![/assets/img/imgLIGA(8).png](/assets/img/imgLIGA(8).png)

## 🚩 Root Flag

`OWASPKL{b0f8c51049b9db31552bda1bd751940a}`
---
title: "OWASP Liga CTF 2026 - The Art of Evasion & Persistence (Boot2Root)"
date: 2026-05-30
categories: [CTF, Writeups, B2R]
tags: [OWASPKL]
description: A challenge featuring IMAP enumeration, credential discovery through email, RiteCMS file upload exploitation, reverse shell access, and Webmin-based privilege escalation to root/
image: /assets/img/OWASPKcover.png
---

## Network Reconnaissance

```
nmap -sCV 192.168.100.120 —vv
```

Results:

```
PORT     STATE SERVICE       REASON          VERSION
21/tcp   open  ftp           syn-ack ttl 128 FileZilla ftpd 1.12.6
| tls-alpn: 
|_  ftp
| ssl-cert: Subject: commonName=filezilla-server self signed certificate
| Issuer: commonName=filezilla-server self signed certificate
| Public Key type: ec
| Public Key bits: 256
| Signature Algorithm: ecdsa-with-SHA256
| Not valid before: 2026-05-26T12:56:16
| Not valid after:  2027-05-27T13:01:16
| MD5:     468e 8004 8e13 c89f a1d5 68db a316 8380
| SHA-1:   821a 65a7 71e3 322e e4cf 6adc 8487 cc01 f554 c8a7
| SHA-256: 727f b199 c711 e550 a384 5a60 ae7b 2f4b e0aa 888b f1c8 c4c0 f014 a1b7 a1e4 68ba
| -----BEGIN CERTIFICATE-----
| MIIBiTCCAS6gAwIBAgIUChA9UkzgJu6aUmjOuO/8eYrdCAowCgYIKoZIzj0EAwIw
| MzExMC8GA1UEAxMoZmlsZXppbGxhLXNlcnZlciBzZWxmIHNpZ25lZCBjZXJ0aWZp
| Y2F0ZTAeFw0yNjA1MjYxMjU2MTZaFw0yNzA1MjcxMzAxMTZaMDMxMTAvBgNVBAMT
| KGZpbGV6aWxsYS1zZXJ2ZXIgc2VsZiBzaWduZWQgY2VydGlmaWNhdGUwWTATBgcq
| hkjOPQIBBggqhkjOPQMBBwNCAAQk/YQvb0stmzEmmH7cxEL1xwrQpmhLiA4Ths4C
| 6pGLl9HYUWQ5PDBumfs8vO5vGYIS3M2ZQ+gY127CwcHINDPboyAwHjAOBgNVHQ8B
| Af8EBAMCBaAwDAYDVR0TAQH/BAIwADAKBggqhkjOPQQDAgNJADBGAiEAmpI4F0HV
| K9MK8MEVcEv69faJYpsOuPi6mGQ5sWs6YjACIQCzE0mzuKDOvOwI0e5ptqSCEcWe
| jb4AqcEr8MlsAOpxdQ==
|_-----END CERTIFICATE-----
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla.
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
80/tcp   open  http          syn-ack ttl 128 Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
135/tcp  open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 128 Microsoft Windows netbios-ssn
443/tcp  open  ssl/http      syn-ack ttl 128 Apache httpd 2.4.58 ((Win64) OpenSSL/3.1.3 PHP/8.0.30)
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.0.30
| http-title: Welcome to XAMPP
|_Requested resource was https://192.168.100.120/dashboard/
| ssl-cert: Subject: commonName=localhost
| Issuer: commonName=localhost
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2009-11-10T23:48:47
| Not valid after:  2019-11-08T23:48:47
| MD5:     a0a4 4cc9 9e84 b26f 9e63 9f9e d229 dee0
| SHA-1:   b023 8c54 7a90 5bfa 119c 4e8b acca eacf 3649 1ff6
| SHA-256: 0169 7338 0c0f 1df0 0bd9 593e d8d5 efa3 706c d6df 7993 f614 1272 b805 22ac dd23
| -----BEGIN CERTIFICATE-----
| MIIBnzCCAQgCCQC1x1LJh4G1AzANBgkqhkiG9w0BAQUFADAUMRIwEAYDVQQDEwls
| b2NhbGhvc3QwHhcNMDkxMTEwMjM0ODQ3WhcNMTkxMTA4MjM0ODQ3WjAUMRIwEAYD
| VQQDEwlsb2NhbGhvc3QwgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAMEl0yfj
| 7K0Ng2pt51+adRAj4pCdoGOVjx1BmljVnGOMW3OGkHnMw9ajibh1vB6UfHxu463o
| J1wLxgxq+Q8y/rPEehAjBCspKNSq+bMvZhD4p8HNYMRrKFfjZzv3ns1IItw46kgT
| gDpAl1cMRzVGPXFimu5TnWMOZ3ooyaQ0/xntAgMBAAEwDQYJKoZIhvcNAQEFBQAD
| gYEAavHzSWz5umhfb/MnBMa5DL2VNzS+9whmmpsDGEG+uR0kM1W2GQIdVHHJTyFd
| aHXzgVJBQcWTwhp84nvHSiQTDBSaT6cQNQpvag/TaED/SEQpm0VqDFwpfFYuufBL
| vVNbLkKxbK2XwUvu0RxoLdBMC/89HqrZ0ppiONuQ+X2MtxE=
|_-----END CERTIFICATE-----
| tls-alpn: 
|_  http/1.1
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_ssl-date: TLS randomness does not represent time
|_http-favicon: Unknown favicon MD5: 6EB4A43CB64C97F76562AF703893C8FD
445/tcp  open  microsoft-ds? syn-ack ttl 128
8080/tcp open  http          syn-ack ttl 128 Apache httpd 2.4.58 ((Win64) OpenSSL/3.1.3 PHP/8.0.30)
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.100.120:8080/dashboard/
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.0.30
|_http-favicon: Unknown favicon MD5: 6EB4A43CB64C97F76562AF703893C8FD
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
MAC Address: 08:00:27:DC:5A:E2 (Oracle VirtualBox virtual NIC)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| nbstat: NetBIOS name: DESKTOP-LGT6HFQ, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:dc:5a:e2 (Oracle VirtualBox virtual NIC)
| Names:
|   DESKTOP-LGT6HFQ<00>  Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|   DESKTOP-LGT6HFQ<20>  Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
| Statistics:
|   08 00 27 dc 5a e2 00 00 00 00 00 00 00 00 00 00 00
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_  00 00 00 00 00 00 00 00 00 00 00 00 00 00
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 29500/tcp): CLEAN (Timeout)
|   Check 2 (port 64311/tcp): CLEAN (Timeout)
|   Check 3 (port 19963/udp): CLEAN (Timeout)
|   Check 4 (port 19959/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-time: 
|   date: 2026-05-30T17:46:23
|_  start_date: N/A
|_clock-skew: 6h59m56s

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 05:47
Completed NSE at 05:47, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 05:47
Completed NSE at 05:47, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 05:47
Completed NSE at 05:47, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 69.57 seconds
           Raw packets sent: 1997 (87.852KB) | Rcvd: 11 (468B)
```

Key findings from the scan:

| Port | Service | Detail |
| --- | --- | --- |
| `21` | FTP (FileZilla 1.12.6) | **Anonymous login allowed** |
| `80` | HTTP (Microsoft IIS 10.0) | Default IIS page |
| `443` | HTTPS (Apache 2.4.58 / PHP 8.0.30) | XAMPP dashboard |
| `445` | SMB | Message signing not required |
| `8080` | HTTP (Apache 2.4.58 / PHP 8.0.30) | XAMPP dashboard |

The most important finding here was that **FTP allows anonymous login** — no credentials needed. This is our initial foothold.

## FTP Anonymous Login

Connected to FTP anonymously and found Windows registry hive files exposed in the FTP root directory which is `SAM`, `SECURITY`, and `SYSTEM`. These are critical Windows credential files.

![/assets/img/imgLIGA2.png](/assets/img/imgLIGA2.png)

After downloading the three files, we used **`Impacket's secretsdump`** to extract the NTLM password hashes from them locally:

![/assets/img/imgLIGA2(1).png](/assets/img/imgLIGA2(1).png)

This dumped multiple NTLM hashes for local accounts including `webadmin`, `ligac`, `jhalim`, `norziana`, `fakhrul`, and others.

### Hash Cracking with Hashcat

We saved only the NT (NTLM) portion of the hashes to a file called `nt_only.txt`, then cracked them using Hashcat with the classic `rockyou.txt` wordlist.

Save hashes to the nt_only.txt file

![/assets/img/imgLIGA2(2).png](/assets/img/imgLIGA2(2).png)

Crack the hashes

![/assets/img/imgLIGA2(3).png](/assets/img/imgLIGA2(3).png)

One hash cracked successfully:

```bash
webadmin : webhead2290
```

So we now have valid credentials.

### SMB Enumeration

With the cracked credentials, used `smbclient` to list available shares on the target:

![/assets/img/imgLIGA2(4).png](/assets/img/imgLIGA2(4).png)

We found several shares: `ADMIN$`, `Backup`, `C$`, `IPC$`, and `Users`. 

Connected to the `Users` share and browsed around:

![/assets/img/imgLIGA2(5).png](/assets/img/imgLIGA2(5).png)

Navigating to `\webadmin\Desktop\`, we found `local.txt` and downloaded it.

Reading the file revealed the local flag:

![/assets/img/imgLIGA2(6).png](/assets/img/imgLIGA2(6).png)

## 🚩 Local Flag

`OWASPKL{b955142d1496bc8d6f0d5b16f014666}`

## NTUSER.DAT Analysis

Still on the SMB session, we pivoted to the `ligac` user's profile. Using **Impacket's smbclient**, we downloaded `NTUSER.DAT` from `ligac`'s home directory.

![/assets/img/imgLIGA2(7).png](/assets/img/imgLIGA2(7).png)

Then used `strings` with `grep` to search for interesting keywords inside the registry hive:

![/assets/img/imgLIGA2(8).png](/assets/img/imgLIGA2(8).png)

This revealed an important path:

```bash
C:\xampp\htdocs\ritecms\media\Loader.exe
```

This tells us **RiteCMS** is installed on the target under the XAMPP web root. RiteCMS typically has its admin panel at `/admin.php`.

### RiteCMS Admin Login

Since we know RiteCMS has /admin.php, we try to browse `http://192.168.100.120:8080/ritecms/admin.php` and got the login page for admin.

![/assets/img/imgLIGA2(9).png](/assets/img/imgLIGA2(9).png)

Logged in using the credentials we cracked earlier:

```bash
username: webadmin
password: webhead2290
```

Login was successful.

### Exploit Research

Searched for known exploits for RiteCMS using `searchsploit`:

![/assets/img/imgLIGA2(10).png](/assets/img/imgLIGA2(10).png)

Several exploits were listed. We examined `php/webapps/50616.txt` (RiteCMS 3.1.0-Remote Code Execution via authenticated file upload):

```bash
searchsploit -x php/webapps/50616.txt
```

![/assets/img/imgLIGA2(11).png](/assets/img/imgLIGA2(11).png)

The exploit shows that RiteCMS uses `.htaccess` to block `.php` uploads, but the filter only checks lowercase extensions. By uploading a file with a **mixed-case extension like `.pHp`**, we can bypass the restriction and get the file executed as PHP.

### Steps from the PoC:

1. Login as admin
2. Go to File Manager
3. Navigate to the `files` directory
4. Upload a PHP webshell with the `.pHp` extension

## Webshell Upload & RCE

We created a simple PHP webshell and uploaded it via the RiteCMS File Manager with the filename `shell.pHp`. The file appeared in the manager under `files/` directory.

![/assets/img/imgLIGA2(12).png](/assets/img/imgLIGA2(12).png)

We verified Remote Code Execution by visiting:

```bash
http://192.168.100.120:8080/ritecms/files/shell.pHp?cmd=whoami
```

Response:

![/assets/img/imgLIGA2(13).png](/assets/img/imgLIGA2(13).png)

`nt authority\system` is the highest privilege on Windows. No need for privilege escalation.

## Reading the Proof Flag

We used `curl` to interact with the webshell from our Kali machine. First, we listed the Administrator's Desktop:

![/assets/img/imgLIGA2(14).png](/assets/img/imgLIGA2(14).png)

Output confirmed `proof.txt` exists. 

We then read it and got the flag:

![/assets/img/imgLIGA2(15).png](/assets/img/imgLIGA2(15).png)

## 🚩 Proof Flag

`OWASPKL{33e9d2bbd6c42bf3b71aefbe7dan1543}`
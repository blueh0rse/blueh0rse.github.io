---
title: 🔵 HTB - Preignition
date: 2023-08-01
categories: [HackTheBox, Machines]
tags: [nginx, gobuster]
---

_Directory Brute-forcing is a technique used to check a lot of paths on a web server to find hidden pages. Which is another name for this? (i) Local File Inclusion, (ii) dir busting, (iii) hash cracking._

`dir busting`

_What switch do we use for nmap's scan to specify that we want to perform version detection_

`-sV`

_What does Nmap report is the service identified as running on port 80/tcp?_

```bash
$ nmap -sV -p- <ip>
...
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2
...
```

_What server name and version of service is running on port 80/tcp?_

`nginx 1.14.2`

_What switch do we use to specify to Gobuster we want to perform dir busting specifically?_

`dir`

_When using gobuster to dir bust, what switch do we add to make sure it finds PHP pages?_

`-x php`

_What page is found during our dir busting activities?_

`admin.php`

_What is the HTTP status code reported by Gobuster for the discovered page?_

`200`

_Submit root flag_

```bash
$ curl <ip>/admin.php
...shows a login form...

$ firefox <ip>/admin.php
admin:admin

6483bee07c1c1d57f14e5b0717503c73
```

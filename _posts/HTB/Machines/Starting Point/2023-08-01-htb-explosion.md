---
title: 🔵 HTB - Explosion
date: 2023-08-01
categories: [HackTheBox, Machines]
tags: [rdp, xfreerdp]
---

# Explosion

What does the 3-letter acronym RDP stand for?

`Remote Desktop Protocol`

What is a 3-letter acronym that refers to interaction with the host through a command line interface?

`cli`

What about graphical use interactions?

`gui`

_What is the name of an old remote access tool that came without encryption by default and listens on TCP port 23?_

`telnet`

Whats is the name of the service running on port 3389 TCP?

```bash
$ nmap -sV -p-
...
PORT      STATE SERVICE       VERSION
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
...
```

What username successfully returns a desktop projection to us with a blank password

`administrator`

Submit root flag

```bash
$ xfreerdp /u:administrator /v:hostname
...
951fa96d7830c451b536be5a6be008a0
```

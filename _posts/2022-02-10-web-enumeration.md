---
title: Web Enumeration Cheatsheet
date: 2022-10-02
categories: [Cheatsheet]
tags: [gobuster]
---

# Gobuster
Performs file and directory brute forcing
````console
gobuster dir -u http://10.10.10.10/ -w /path/to/wordlist.txt	
````

DNS resolution brute forcing
````console
gobuster dns -d website.com -w /usr/share/SecLists/Discovery/DNS/namelist.txt
````

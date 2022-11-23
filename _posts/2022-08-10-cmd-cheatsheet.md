---
title: Commands Cheatsheet
date: 2022-10-02
categories: [Cheatsheet]
tags: [smb]
---

# SMB
List SMB Shares
````console
smbclient -N -L \\\\10.10.10.10 	
````

Connect to an SMB share as anonymous
````console
smbclient \\\\10.10.10.10\\share
````

Connect to an SMB share as user bob
````console
smbclient -U bob \\\\10.10.10.10\\share
````

# Telnet

# curl

# SSH
````console
ssh user@ip
````

# nmap
Basic nmap scan
````console
nmap 10.10.10.10
````

Run an nmap script scan on an IP
````console
nmap -sV -sC -p- 10.10.10.10 	
````

List various available nmap scripts
````console
locate scripts/citrix 	
````

Run an nmap script on an IP
````console
nmap --script smb-os-discovery.nse -p445 10.10.10.10 	
````

Grab banner of an open port
````console
netcat 10.10.10.10 22 	
````

Save scan result
````console
nmap -oA file_name 10.10.10.10
````

# SNMP
Scan SNMP on an IP
````console
snmpwalk -v 2c -c public 10.10.10.10 1.3.6.1.2.1.1.5.0 	
````

Brute force SNMP secret string
````console
onesixtyone -c dict.txt 10.129.42.254 	
```` 



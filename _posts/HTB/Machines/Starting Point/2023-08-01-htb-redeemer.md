---
title: 🔵 HTB /  Redeemer 
date: 2023-08-01
categories: [HackTheBox, Machines]
tags: [redis]
---

# Redeemer

_Which TCP port is open on the machine?_

```bash
$ nmap -sC -sV <ip>
no port open

$ nmap -sC -sV -p- <ip>
...
PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 5.0.7
```

_Which service is running on the port that is open on the machine?_

`Redis`

_What type of database is Redis?_

`In-memory Database`

_Which command-line utility is used to interact with the Redis server?_

`redis-cli`

_Which flag is used with the Redis command-line utility to specify the hostname?_

`-h <hostname>`

_Once connected to a Redis server, which command is used to obtain the information and statistics about the Redis server?_

```bash
$ redis-cli -h <hostname> -p <port>
hostname:port> info
```

_What is the version of the Redis server being used on the target machine?_

`5.0.7`

_Which command is used to select the desired database in Redis?_

`select`

_How many keys are present inside the database with index 0?_

```bash
hostname:port> info
...
# Keyspace
db0:keys=4,expires=0,avg_ttl=0
...
```

_Which command is used to obtain all the keys in a database?_

`keys *`

_Submit root flag_

```bash
hostname:port> keys * 
1) "temp" 
2) "stor" 
3) "numb" 
4) "flag" 
hostname:port> keys 4 
(empty array) 
hostname:port> keys flag 
1) "flag" 
hostname:port> get flag 
"03e1d2b376c37ab3f5319922053953eb"
```

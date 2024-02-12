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
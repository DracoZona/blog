---
layout: post
title:  "SQL Injection on gRPC Endpoint"
author: sal
categories: [ SQL Injection, HTB ]
image: assets/images/htb-pc.png
featured: true
---

##### Introduction
This is a machine from HTB called "PC". It focuses on exploiting a vulnerability in the **gRPC** endpoint to gain foothold into the machine and eventually perform privilege escalation techniques.

##### Walkthrough
First, I have performed nmap scan to check the available open ports.
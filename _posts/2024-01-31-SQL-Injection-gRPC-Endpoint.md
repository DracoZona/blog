---
layout: post
title:  "SQL Injection on gRPC Endpoint"
author: dz
categories: [ SQL Injection, HTB ]
image: assets/images/htb-pc.png
featured: true
---

This is a machine from HTB called "PC". It focuses on exploiting a vulnerability in the **gRPC** endpoint to gain foothold into the machine and eventually perform privilege escalation techniques.

##### Introduction
First, I have performed nmap scan to check the available open ports.
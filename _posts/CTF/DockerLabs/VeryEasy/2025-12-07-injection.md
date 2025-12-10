---
title: "DockerLabs: Injection"
date: 2025-12-07
categories: [CTFs, Dockerlabs]
tags: [DockerLabs, Dockerlabs Very Easy]
---

Machine by [El Pingüino de Mario](https://www.youtube.com/channel/UCGLfzfKRUsV6BzkrF1kJGsg)

---

### 1. nmap scan

Enumerate the machine and get all the important information.

By using the next command:

`nmap -p- -sV -p- VICTIM_IP`

We can see 2 open ports with no version vulnerability

![](/assets/img/ctf/dockerlabs/very_easy/injection/injection_01.png)

### 2. SQL Injection

After visiting the website we can see a login form, where, gibben the machine name i'll try sql injection with `'or'1'=1` on both user and password field:

![](/assets/img/ctf/dockerlabs/very_easy/injection/injection_02.png)


We can see that it worked and we get the user and password in plain text:

![](/assets/img/ctf/dockerlabs/very_easy/injection/injection_03.png)

<br>

### 3. SSH Access

We can see that the user and password is valid for the ssh service:

![](/assets/img/ctf/dockerlabs/very_easy/injection/injection_04.png)
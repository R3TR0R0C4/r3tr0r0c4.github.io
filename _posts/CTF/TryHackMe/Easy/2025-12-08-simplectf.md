---
title: "TryHackMe: Simple CTF"
date: 2025-12-08
categories: [CTFs, TryHackMe, Easy]
tags: [TryHackMe, TryHackMe Easy]
---

Machine by [MrSeth6797](https://tryhackme.com/p/MrSeth6797)

Tryhackme [link](https://tryhackme.com/r/room/easyctf)

---

## Tools Used:<!-- omit in toc -->

- Kali Linux
- NMAP
- Dirb


---

### 1. Nmap Scan

Doing a nmap scan reveals a webserver, ftp server and ssh on port 2222:

![](/assets/img/ctf/tryhackme/easy/simplectf/simplectf_01.png)

<br>

### 2. Dirb Scan

We can see a index, wich turns into a default apache installation page, a robots.txt with nothing interesting in it, and a "cms made simple" installation:

![](/assets/img/ctf/tryhackme/easy/simplectf/simplectf_02.png)

<br>

### 3. CMS Made Simple Exploitation

As we can see at the bottom of the page, the version of the Simple CMS is 2.2.8:

![](/assets/img/ctf/tryhackme/easy/simplectf/simplectf_03.png)

<br>

After googling `Simple CMS 2.2.8`, we can see this exploit on [exploit-db](https://www.exploit-db.com/exploits/46635):

![](/assets/img/ctf/tryhackme/easy/simplectf/simplectf_04.png)

<br>


The exploit consists of a python script that we'll execute with the options `--url=http://Server-IP/simple/` and `--wordlist=/usr/share/wordlists/rockyou.txt`:

![](/assets/img/ctf/tryhackme/easy/simplectf/simplectf_05.png)

<br>

After executing it, this  is the result:

![](/assets/img/ctf/tryhackme/easy/simplectf/simplectf_06.png)

As we can see the provided username and password seems to not be correct:

![](/assets/img/ctf/tryhackme/easy/simplectf/simplectf_07.png)

<br>

### 4. SSH login

I've tried to use hydra to get the ssh password with the previous username, as we can see we get a password, `secret`:

![](/assets/img/ctf/tryhackme/easy/simplectf/simplectf_08.png)

<br>

Let's try to login with ssh:

![](/assets/img/ctf/tryhackme/easy/simplectf/simplectf_09.png)

### 5. User flag

In the mitch home folder is the first flag:

![](/assets/img/ctf/tryhackme/easy/simplectf/simplectf_10.png)

### 6. Privilege escalation

With `sudo -l` we'll check if there is any file or executable to wich we have root access without a password, as we can see we've got access to vim as root:

![](/assets/img/ctf/tryhackme/easy/simplectf/simplectf_11.png)

<br>

Let's execute vim with sudo:

![](/assets/img/ctf/tryhackme/easy/simplectf/simplectf_12.png)

<br>

And execute bash:

![](/assets/img/ctf/tryhackme/easy/simplectf/simplectf_13.png)

<br>

As we can see we're root:

![](/assets/img/ctf/tryhackme/easy/simplectf/simplectf_14.png)

<br>

### 7. Root flag

Let's get the root's flag:

![](/assets/img/ctf/tryhackme/easy/simplectf/simplectf_15.png)

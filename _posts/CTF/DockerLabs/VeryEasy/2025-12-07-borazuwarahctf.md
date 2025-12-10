---
title: "DockerLabs: BorazuwarahCTF"
date: 2025-12-07
categories: [CTFs, Dockerlabs, Very Easy]
tags: [DockerLabs, Dockerlabs Very Easy]
---

Machine by BorazuwarahCTF

---

### 1. nmap scan

Enumerate the machine and get all the important information.

By using the next command:

`nmap -p- -sV -p- VICTIM_IP`

We can see 2 open ports with no version vulnerability

![](/assets/img/ctf/dockerlabs/very_easy/borazuwarahctf/borazuwarahctf_01.png)

<br>

### 2. Visiting the site

After visiting the site we can see a kinder egg with no apparent way to advance, there aren't any hidden clues on the site's html source code either.

![](/assets/img/ctf/dockerlabs/very_easy/borazuwarahctf/borazuwarahctf_02.png)

<br>

### 3. Web directory enumeration

Using dirb and enumerating the directories we can't see anything else other than the index:

![](/assets/img/ctf/dockerlabs/very_easy/borazuwarahctf/borazuwarahctf_03.png)

<br>

### 4. Image analysis

Seeing that the site doesn't have any apparent way to advance let's do some steganography on the image

Using steghide there is a txt file:

![](/assets/img/ctf/dockerlabs/very_easy/borazuwarahctf/borazuwarahctf_04.png)

Nothing useful here...

![](/assets/img/ctf/dockerlabs/very_easy/borazuwarahctf/borazuwarahctf_05.png)

We can look into the image metadata with exiftool, and we can see a user:

![](/assets/img/ctf/dockerlabs/very_easy/borazuwarahctf/borazuwarahctf_06.png)

### 5. SSH Bruteforce

Using hydra we can get the user's password of ssh and with the rockyou wordlist:

![](/assets/img/ctf/dockerlabs/very_easy/borazuwarahctf/borazuwarahctf_07.png)

We can see that the login to the ssh service is successful:

![](/assets/img/ctf/dockerlabs/very_easy/borazuwarahctf/borazuwarahctf_08.png)

<br>

### 6. Privilege escalation

With `sudo -l` we can list the user's permissions of sudoer of file/programs, we see that we can execute bash with sudo and no password and successfully get root access:

![](/assets/img/ctf/dockerlabs/very_easy/borazuwarahctf/borazuwarahctf_09.png)

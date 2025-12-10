---
title: "DockerLabs: AguaDeMayo"
date: 2025-12-07
categories: [CTFs, Dockerlabs, Easy]
tags: [DockerLabs, Dockerlabs Easy]
---

Machine by The Hackers Labs

---

### 1. nmap scan

Enumerate the machine and get all the important information.

By using the next command:

`nmap -sV -p- VICTIM_IP`

We can see 2 open ports with no version vulnerability

![](/assets/img/ctf/dockerlabs/easy/aguademayo/aguademayo_01.png)

<br>

### 2. Site vulnerability

Using gobuster, enumerate the website to list its pages. This reveals an `images` folder and the `index` page:

![](/assets/img/ctf/dockerlabs/easy/aguademayo/aguademayo_02.png)


When visiting the site, it only displays the default Apache installation page:

![](/assets/img/ctf/dockerlabs/easy/aguademayo/aguademayo_03.png)

However, nspecting the page source uncovers an unusual code:

![](/assets/img/ctf/dockerlabs/easy/aguademayo/aguademayo_04.png)


This is identified as [BrainFuck code](https://en.wikipedia.org/wiki/Brainfuck). Decoding it results in the string: `bebeaguaqueessano`

![](/assets/img/ctf/dockerlabs/easy/aguademayo/aguademayo_05.png)

However it's not a valid directory or page:

![](/assets/img/ctf/dockerlabs/easy/aguademayo/aguademayo_06.png)

<br>

### 3. Images folder

After visiting the images folder we can see a `agua_ssh.png` file:

![](/assets/img/ctf/dockerlabs/easy/aguademayo/aguademayo_07.png)

The file does not seem to provide any immediately useful information:

![](/assets/img/ctf/dockerlabs/easy/aguademayo/aguademayo_08.png)

<br>

### 4. SSH Login

After trying some combinations, we can see that the user for ssh is `agua` and the password `bebeaguaqueessano`:

![](/assets/img/ctf/dockerlabs/easy/aguademayo/aguademayo_09.png)

<br>

### 5. Privilege escalation

Running `sudo -l` reveals that the user has permission to execute bettercap with sudo privileges without providing a password:

![](/assets/img/ctf/dockerlabs/easy/aguademayo/aguademayo_10.png)

Execute bettercap with sudo:

![](/assets/img/ctf/dockerlabs/easy/aguademayo/aguademayo_11.png)

Within bettercap, type `!` to execute code as in a shell, gaining root access:

![](/assets/img/ctf/dockerlabs/easy/aguademayo/aguademayo_12.png)

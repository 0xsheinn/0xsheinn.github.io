---
published: false
---
# [](header-3) Introduction@blunder:~$

| Column            | Details          |
|:------------------|:-----------------|
| Name          	| Blunder     	   |
| IP            	| 10.10.10.191     |
| Points 			| 30			   |
| Os           		| Linux			   |
| Difficulty		| Easy	 		   |
| Creator			| egotisticalSW    |
| Out on     		| 30 May 2020	   |


# [](header-2)Recon

# [](header-3)Nmap Scanning

```
┌─[root@cyb3rhoL1c]─[~/Documents/htb/writeup/blunder]
└──╼ #nmap -sC -sV -oN nmap 10.10.10.191
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-15 23:22 +0630
Nmap scan report for blunder.htb (10.10.10.191)
Host is up (0.32s latency).
Not shown: 998 filtered ports
PORT   STATE  SERVICE VERSION
21/tcp closed ftp
80/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Blunder
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Blunder | A blunder of interesting facts

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.17 seconds

```

It's only port 80 `open`,=

![](1)

Let's directory scan

# [](header-3)Directory Scan

```
┌─[✗]─[root@cyb3rhoL1c]─[~/Documents/htb/writeup/blunder]
└──╼ #dirsearch -u 10.10.10.191 -w /opt/wordlists/directory-list-2.3-medium.txt -e *

 _|. _ _  _  _  _ _|_    v0.3.9
(_||| _) (/_(_|| (_| )

Extensions: 1.png | HTTP method: get | Threads: 10 | Wordlist size: 220521

Error Log: /root/Downloads/dirsearch/logs/errors-20-10-15_23-24-39.log

Target: 10.10.10.191

[23:24:55] Starting: 
[23:24:57] 200 -    7KB - /       
[23:24:58] 200 -    3KB - /about 
[23:25:04] 200 -    7KB - /0       
[23:25:12] 301 -    0B  - /admin  ->  http://10.10.10.191/admin/
```

I got some interesting directories ,`/about` and `/admin`

> /about

![](2)

> /admin

![](3)

It's login panel , from the directory scan , I got anther 2 files `/robots.txt` and `/todo.txt`

> /robots.txt

![](4)

> todo.txt

![](5)



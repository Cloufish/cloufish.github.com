---
title: Why should you root your own android device?
date: 2021-03-13 06:46:00 +0100
categories: [Mobile Bug Bounty, Preparing Device]
tags: [mobile bug bounty]
lang: en
---
# What is rooting?

It is essentially changing permissions that the app has when it comes to accessing other applications' data and also yours. I'll explain it further :)

## The state of your device before rooting

### Dalvik VMs

![dalvik](https://imgur.com/coQAIcq.jpg)

As you can see, every app has It's own 'Virtual Machine' (sandbox) which limits the Apps access to the filesystem.
App1 is only able to access the data from directory ```/data/../<application_name>```

With rooted device however, the permissions are not enforced. Meaning that apps could gain access to other apps' data.

Also, you can clearly see that system process also has its own VM, and so to access these system files we need to root our device.

## Why do we need to remove these permissions?

Basically to gain bigger control over the device.

One might say that it's stupid to remove permissions, because after all we examine the impact of a vulnerability based on the default state, right? And it's true, in the end we'll be estimating impact based on the default setup, but **to examine the application's behavior more deeply, wee need full access.**
Many pentesting-tools also require root access, because how would they access other app's data? How would they analyze It's actions dynamically?


## Alternatives to rooting your device

With android debug bridge (```adb```) debugger you could gain root access to the applications data without the need of rooting your device. However, you wouldn't have access to every system file

All you have to do is enabling adb debugging in Developer Options in your Android settings and that's it.

**But what about other tools** that only work with root access? Examples of such might be ```frida```, or any other software that let's us hook to a running application.
## Dangers of rooting

- Giving the app's ability to gain root access may be destructive, if you use this device for a daily tasks, **you shouldn't use it that way**


## Should you root your device?
- I don't want to give statement that rooting is a necessary thing to do. It's gives the comfort and removes headache of working in a limited environment. **It's makes things less problematic**
- I recommend you rooting it, but **only if won't use it on a daily tasks.**

---
title: Preparing your PHYSICAL Android device for Bug-Bounty Part 1
date: 2021-03-02 06:46:00 +0100
categories: [Mobile Bug Bounty, Preparing Device]
tags: [mobile bug bounty]
lang: en
---

## Requirements:
1. You have your own PHYSICAL Android Device, **that you don't use on a daily basis**. It can be an old phone, the only thing that matters is that you don't use it on a daily basis
1. Knowing the basic parameters of your device e.g:
    - Phone model
    - CPU Architecture that your device has (ARM/ARM64/x86/x86_64)
1. [Odin Flash Tool](https://odinsoft.site/download/) installed
1. Willingness to Google things.

> That's basically it!

## What we will cover for now:
1. **Installing TWRP** (Custom Recovery to install essentially zip files, and also custom ROMs on a 'root' level)
1. **Installing custom firmware** that will be lightweight, but also would allow you to install newer versions of software that you'll be testing
1. **Installing GApps**, so that you could install software from *Google Play*
1. **Rooting your device**
1. **How to make backups** of Android system so that you can always restore if something would go wrong.

## Installing TWRP

### What is TWRP?

As I stated it is a custom recovery for Android device.

### Why are we installing it?

The stock/normal recovery in your Android doesn't allow you to do that many things. Stock recovery is mostly just about... doing factory reset and then recovering from existing recovery-image.

But **TWRP serves many other features**:
- Backups of partitions
- Restoring these backups
- Custom Firmware Installation! <- Yay!
- File deletion
- Terminal access
- Theme Support
- Possible decryption support

### Where do I get one?
You need to go to https://twrp.me/Devices/ and then search for your device model and download the file.

### Installation.

1. Turn off your phone
1. Open up Odin3 Flash Tool (Hope you installed it :P)
1. Boot your phone into **Download mode**, it's mostly pressing a combination of your phone physical keys, but that all depends on your model, you need to google it :/
1. In your Odin:

![Odin](https://i.imgur.com/yDK6Qtt.png)
(1.) You should click on 'AP' button, then choose your TWRP image.
(2.) You should also see that one of these fields signalizes that your device is connected.
(3.)  If you've done these two exhausting steps and everything *seems* correct, press Start button and don't disconnect your phone.

You've basically just installed TWRP! Congrats!

## Custom Firmware
### Why should you install it?
- Custom firmware can provide extra functionalities
- Also they're free of bloated stock software that Google, Samsung on any other company inserts into your device. *Saves a lot of disc space!*
- It is also better optimized with kernel 'enchantments'

However, if you feel completely comfortable, or you have a relatively new phone that is just... fast! Then you don't necessarily install custom firmware
### What custom firmware should I use?
 I personally recommend those from **SlimROMs** organization. They support a vast majority of Samsung devices, and are probably the most lightweight out there.

They do not provide download links on their website. So you need to Google the phrase "[Your PHONE Model] SlimROMs". You'll probably stumble upon XDA forum. This is one of the most trusted places to download the image.

 ### Installing SlimROMs (Or any other)
 1. Turn off your phone again, and boot it into **Recovery Mode**
 1. From your computer upload/push the custom ROM image into your phone's sdcard
 1. Click on the [Install] button, choose custom ROM zip file and install it.

 You've now installed custom ROM, but that's not enough **please don't quit out of your TWRP** though nothing would happen if you do, you would need to enter there again for the next step!

 ## Installing GApps.
 ### What is it?
 GApps essentially gives you ability to install Google Apps for your newly installed custom firmware. Yes - these apps are not shipped with custom ROMs because of some licensing issue. The solution that I provide is probably the easiest one to have your Google Apps working! :P

 ### Installing it

Head to https://opengapps.org/ and then choose the CPU architecture that your phone uses and also Android version **that the custom firmware has**

The procedure is the same as with custom ROMs:
1. You upload the file to sdcard
1. Enter TWRP
1. Install the zip file

## Rooting your device
### Miscalculations...
From now there may be difficulties/miscalculations!.

If you have Samsung phone, then you have easy one, because there's CF-Auto-Root which enables SuperSU (SuperUser).

[XDA Forum Post](https://www.xda-developers.com/root/) contains a great list of devices that can be rooted (Though Samsung is still the easiest don't lose hope!) ->.

**I'll be sticking with CF-Auto-Root.** I do not have the required knowledge for now to know every method, but you can read about it! Likewise, I'll read it myself later too and probably update this section or creating a new post for the purpose of explaining.
### 'Installing it'
In the case of CF-Auto-Root, you need to...
1. Download the .tar.md5 from this site -> https://autoroot.chainfire.eu/.
1. If it's zipped, unzip it!
1. Then boot up into **Download Mode**
1. And with Odin, the same as in the case of TWRP, select the file and click START
1. Don't turn off your device.

If you're able to use CF-Auto-Root then congrats, and if your phone boots and executes exploit, then you have rooted Android phone!

## Things to do after doing all these steps
 You should do basic configuration of the device after it boots up for the first time.
### Install TWRP (AGAIN)
After rooting your device you might stumble into issue that you don't have TWRP installed now. If so, then you should install it again with the same steps as we had before!
### Doing Backups
If you have done all these steps, you might see that they're relatively easy (Don't worry if you don't understand it now), but also time-consuming.

I recommend you doing backups very often. I don't think you would need that much sdcard space free, so you can use this space to store your backup images.

You can also store these backup images on your computer, Google Drive or anywhere else.

To do simple backups you'll need:
1. TWRP Custom Recovery (that you probably have)
1. In TWRP there'll be an option to do a backup - that easy!
1. If you needed to restore the image just use **restore** option in TWRP

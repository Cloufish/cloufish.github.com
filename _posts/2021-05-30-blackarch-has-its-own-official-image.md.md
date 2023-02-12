---
title: Blackarch has it's own official docker image!
date: 2021-05-30 08:46:00 +0100
categories: [Docker, blackarch-zsh]
tags: [docker, pentesting lab]
lang: en
---

## You can find it [HERE](https://hub.docker.com/r/blackarchlinux/blackarch)


## That's amazing news.
Having an official and stable container will make it much easier to develop other blackarch containers, and I wish there was one when I was creating my container image.

## Is blackarch-zsh useless now?
I don't think that.

### The size of blackarchlinux official vs blackarch-zsh-minimal
The official blackarch provides the most minimal setup possible, with mainly blackarch keyring imported.

blackarch-zsh-minimal provides some tweaking, imported configs, tmux sessions set up, go, python setup and many other things!

While this minimalistic approach is completely fine, an average or even beginner Linux user trying out using blackarch official container may be confused, because of lack of sudo command, and many others commands that he uses on a daily basis.

**But let's face the truth**:
- blackarchlinux official has size of **400MB**
- blackarch-zsh-minimal has size of **2.1GB**

So I have to learn a lesson, and at least reduce it's size to 1GB.

### I've created blackarch-zsh without being and expert of archlinux
- So without a doubt, they did many things better, and while I was creating my image, I did ask a lot of questions to them

## But I still hope, that the workarounds in my Dockerfile will help.
And maybe give you a hint of how to solve your problems when creating docker-image


## Summary
This blog-post is small, I know. I mainly wanted to share with you an information that there's a blackarch official image that you can use.
Cheers :)

---
title: Learning on how to build a modern WebApp - MEAN STACK
date: 2021-04-05 06:46:00 +0100
categories: [DevSecOps, Programming Projects]
tags: [mean, stack, angularjs, nodejs, express, mongodb, programming]
lang: en
---

I've recently decided that it is a great idea to learn the 'basics' (from my perspective) of modern WebApp development. This post won't be some kind of guide - just my thoughts :)

## Why learning WebApp Programming?

### from a Pentester's perspective
- It helps you better understand the WebApp, and thus you'll have more chance of catching some mistakes, bad practices (maybe)
- In my opinion, it's way better to learn the relationships between Client and the Server Side (Even that there exists concept like that), and building security knowledge upon that, than to after months of confusion finally realize it, just with ctfs, bugbounty (This statement may be abstract for some people, though for a beginner it takes some time to acquire this knowledge only with ctfs, reading Security stuff)
- It gives you basically a free, your own lab where you can test basic OWASP 10 Vulnerabilities. You can also see by that if these frameworks are secure by default or not
- Learning JavaScript in my opinion can only be accomplished by doing WebApp projects. If you know them, then vulnerabilities that require code review/analysis (DOM Vulnerabilities for example) are open!


### From a DevSecOps Perspective
- Understanding on how to code gives you some insight on Developer's work. That there are many things to miss for him
- If you gain this knowledge, you'll probably gain better ability of communicating with Developer Teams. After all AppSec culture is well-established through Security Team and Developers
- Also you'll probably better evaluate on how to implement Security practices into developer's workflow
- If You'd want to implement security by default (through creating wrappers around *potentially* unsafe libraries), you'll have to learn programming anyway. Understanding most popular web stacks and the one that your developers use is the first step to take!

## What course did I choose?

Basically that one -> [MEAN CRASH COURSE](https://www.youtube.com/watch?v=1tRLveSyNz8&pbjreload=101)

There're tremendous amounts of great courses like these. There's no need in spending money on courses and Bootcamp (if you don't want to be programmer)

This course was about building a WebApp on the MEAN Stack
## What is Stack - what does it even mean? You mean StackOverflow, right?

Noo... The terminology 'Stack' signalizes on what technologies the WebApp is built.

For Example - the one that I made from the course is based on the **MEAN** Stack. Which stands for
(MongoDB, Express, AngularJS, NodeJS)

These technologies together create an excellent ecosystem for building a modern web-app

## My WebApp Developing Background:

Basically none. In high school I was creating a lot of projects with PHP code that would add a new row/data into MySQL database, also some JavaScript to change the color of the buttons, CSS and HTML.

## My impression/obstacles

It's so amazing to see how modern frameworks work. In this video many of stuff that was written was in typescript instead of JavaScript.
Adding new elements to the Posts object is so abstract to me! Also, ```@Output()``` and ```@Input()``` function.
calls seem peculiar!.



## Other thoughts

- This course though is a bit outdated, it has almost 3 years and much stuff has changed. For example now you have to specify a deeper path to import a new module, not just doing some sort of ```import * from``` as many do (which is a bad practice!) in Python
- Even though I didn't understand everything, I've accepted this and nonetheless I concentrated on understanding the most of it
- To my disappointment It only touched briefly the idea of NodeJS, Express and MongoDB and explained only the concepts.

That's basically it from me today :) I'll probably be doing more of such courses, but next time the newer one, and also more complete (I've already found about [MEARN Stack](https://youtu.be/ktjafK4SgWM) which I think will be way better!)

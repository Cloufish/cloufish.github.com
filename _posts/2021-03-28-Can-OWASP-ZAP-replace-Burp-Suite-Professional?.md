---
title: Can OWASP ZAP replace Burp Suite Professional?
date: 2021-03-28 06:46:00 +0100
categories: [Web Bug Bounty, Tools]
tags: [owasp, zaproxy]
lang: en
---

## The short answer - NO :)

Burp Suite offers a tremendous scope of functionalities in one of his app implementation, it has also funds from the paid version to develop this tool more and more, so I don't think It'll ever be replaced with OWASP ZAP.

## The question should be - What functionalities of Burp Suite Professional can I have for free?

I got this idea for a new Hacksplained's video called [Burp Suite Professional Features For Free (Pimp your Community Edition)](https://www.youtube.com/watch?v=FMGa6xBzNck)

It is really awesome, I really recommend you checking it out!

But I realized something... that he didn't limit himself only to using Burp Extensions, but he used functionalities outside the Burp Suite program - that might be obvious to some of you, did you really think that there's a free version/alternative of Burp Collaborator other than hosting your own OOB(Out of Band) server? I did not!

These all external functionalities that he has shown could also be used to **PIMP THE OWASP ZAP TO WORK AS BURP SUITE PRO**

It is to be honest the same scenario. Because OWASP ZAP offers a free Web-Vulnerabilities-Scanner that in Burp Suite cannot get acquired for free but is also extremely handy!

## So let's analyze this video bit by bit, but in OWASP ZAP's case

### Burp Search in OWASP ZAP

That is fairly simple, in OWASP there's a Search Tab:

![Search-Tab](https://imgur.com/BDM2ySH.png)

### Burp Suite CSRF PoC

You just use external site/script [csrf-poc-generator](https://github.com/merttasci/csrf-poc-generator)

Just as it was mentioned in Hacksplained's video

### Burp Collaborator for ZAP

Again, just as was mentioned in the video, we can use [RequestBin](https://requestbin.net/)

### Burp Intruder for ZAP

I've covered the topic of Fuzzing in OWASP ZAP [On My Other Blogpost](https://cloufish.github.io/blog/posts/OWASP-ZAP-as-a-great-fuzzing-tool/).

The final conclusion that I pointed there is that we need to use ZAP's fuzzing [scripts](https://github.com/zaproxy/community-scripts/tree/master/httpfuzzerprocessor), so it's the same case as in Turbo Intruder.

So this section is I think a win for Burp Suite Pro because nonetheless, you would have to learn Python/JS scripting to use it for free.

### Burp Suite Scanner for ZAP

The active scanning capabilities that are in ZAProxy are awesome! I don't want to make statements that these are better than in Burp Suite because I did not use the Scanner of Burp to make a non-biased comparison, but many aspects that make Burp Scanner great are also in ZAProxy!

With many built-in scanning rules, you'll probably stumble upon many vulnerabilities that you didn't know they've existed!

ZAP scanner provides a good insight into how requests are constructed, what headers are missing and potentially make the app vulnerable, how the CORS is configured, Information Disclosures.

![Scanner](https://imgur.com/cb3UeXU.png)

## Summary

There are many functionalities that OWASP ZAP provides, e.g. [HUD](https://www.youtube.com/watch?v=7WL-emt5PDc) (That's extremely awesome function!) and [Report Generator](https://www.youtube.com/watch?v=kD540gUWJ3I) which (I think) are not in Burp Suite

But there are probably other functions of Burp Suite that are not included in ZAProxy! And that's fine!

I prefer though to use OWASP ZAP and to complement the lacking functionalities of it with the things mentioned in Hacksplained's video!

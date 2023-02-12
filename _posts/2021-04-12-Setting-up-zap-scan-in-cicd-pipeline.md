---
title: Setting up ZAP Scan in CI/CD pipeline
date: 2021-04-12 06:46:00 +0100
categories: [DevSecOps, CI/CD]
tags: [owasp, zaproxy, ci, devsecops]
lang: en
---

## What is CI/CD pipeline?
To put it simply, it is a pathway to deployment, in a fast way in respect with Agile manifesto.
Every pull request on GitHub should be checked in order to check if it's working accordingly and prepare it for production area.

There are various CI/CD segments like managing deployment architecture, doing Jest tests, but to us - security people We should focus on:

- Application automation security testing - **This one we'll cover today**
- Automating Vulnerability Management
- Eliminating easy to check vulnerabilities in code, also in pull requests. It could be functions such as ```eval()```, or more complex stuff like lack of JWT token validation and also enforcing the use of secure functions by default.
- Monitoring and collecting data - Without this data, automation is way harder

## Integrating ZAP Action into CI pipeline:
This will provide a straightforward step-by-step tutorial, also explaining every bit of code, feel free to comment if there are any other things you don't understand!

> **IMPORTANT** - Although It won't cover every basic on Github Actions, to get familiar with the basics I recommend this [GitHub's interactive lab](https://lab.github.com/githubtraining/github-actions:-continuous-integration)

### #1 Finding Vulnerable App on which we could build the ZAP tests!

In the -> [Awesome DevSecOps](https://github.com/devsecops/awesome-devsecops#training) repository We can find a pretty good list of Vulnerable Test Targets.
I'll be choosing [NodeGoat](https://github.com/OWASP/NodeGoat). Let's **fork** it!

After We've forked it, we **need** to enable Issues for this forked repo. This is as simple as checking the option in the Settings tab:
![issues](https://imgur.com/Zk58nHJ.png)

This is necessary because ZAP output will be shown as a newly created issue. Without these options checked the ZAP just won't finish its work.

### #2 Creating workflow.yml file
The name 'workflow.yml' is given here as an example, it can be named whatever you want. However, it needs to be in ```.github/workflows``` directory

### #3 Building the Application in a local environment

Because ZAP tests run on a **live** target, we need to first run the app in the local GitHub environment. This can be done using ```docker-compose``` with ```--detached``` flag, and of course for that we need Dockerfile.

Fortunately for us - this repo also gives us a Dockerfile and docker-compose.yml file in the projects directory.

Let's edit the workflow file to contain following lines:

```yml

name: Security Checks
on: [push]

jobs:
  test:
    name: OWASP ZAP SCANS
    runs-on: ubuntu-latest

    steps:
       - uses: actions/checkout@v2
         with:
          ref: master
       - name: Building Docker Node-Goat Image
         run: docker-compose build
       - name: Launching the app
         run: docker-compose up --detach
```
This would launch the app.
The ```- uses: actions/checkout@v2``` line gives us the access to the source code, it's like locally running git clone (though it's probably more complicated than that).

### #3 Integrating ZAP tests

We'll do that by adding the next action.

When editing the workflow file you may have noticed that you have a 'Marketplace' on the right side. We'll use it now and search for *'ZAP'*, we'll get 2 results that:
1. OWASP ZAP Full Scan
2. OWASP ZAP Baseline Scan.

The first one is more time-consuming scan, however it covers much bigger scope, and it's an active scan. I think that It should be run only 1 times in week or even less.

OWASP ZAP Baseline Scan however is ideal for CI/CD pipeline. It's fast and won't interfere that much in developers branch merging. It'll check passively for things like CSP Policies, Headers which are not set, Cookie problems.

![ZAP](https://imgur.com/PD31qio.png)

We need to copy this snippet into our workflow file.

Then, you'll probably need to fix the indentation - The ```.yaml``` format is very strict on these, and also GitHub has some other conventions.

Defining the values of these parameters are pretty straightforward - you just have to write it after the colon sign

I think that every parameter is easy to understand. The one that might cause a problem is ```token: ```.
You just need to provide it a value of ```${{ secrets.GITHUB_TOKEN }}```, so it'll be something like this -> ```token: ${{ secrets.GITHUB_TOKEN }}```
The ```target: ``` should be pointing to a localhost:4000 - The port set in the Dockerfile.

The workflow file that I've come around was:

```yml

name: Security Checks
on: [push]

jobs:
  test:
    name: OWASP ZAP SCANS
    runs-on: ubuntu-latest

    steps:
       - uses: actions/checkout@v2
         with:
          ref: master
       - name: Building Docker Node-Goat Image
         run: docker-compose build
       - name: Launching the app
         run: docker-compose up --detach

       - name: OWASP ZAP
         uses: zaproxy/action-baseline@v0.4.0
         with:
           # Target URL
           target: "http://localhost:4000"
           fail_action: false
           token: ${{ secrets.GITHUB_TOKEN }}
           issue_title: Security Tests

```
And that's basically it. The new issue will be created:
![issues2](https://imgur.com/rBENnhL.png)

However, there are caveats:

## Caveats
- The ZAP scan **will not replace** the human eye. You **should** examine every one of these issues, and If It relates to your application! There's no reason to point out the nonexistence of a Header mitigating Clickjacking, if there are no forms to submit
- This also will mostly remove only the long-hanging fruits. You should still hire a Pentester, create Bug Bounty Program to search for more severe (once you eliminate these long-hanging ones)
- The use of ```rules_file_name:``` is highly advisable to remove those fail-positives I was writing about in the previous points.

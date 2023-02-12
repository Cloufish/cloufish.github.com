---
title: Concept of Preventing Insufficient Logging & Monitoring
date: 2021-04-26 06:46:00 +0100
categories: [DevSecOps, Monitoring]
tags: [devsecops, monitoring]
lang: en
---

## Intro
Security Breaches happen, and also you should be prepared for them to happen, expect them! With this mindset you can minimize the impact of the breach.

With **logging,** we can be better prepared for the breach. I'll explain this topic more deeply in this post :)

Also let's remember that Insufficient Logging & Monitoring **is in OWASP 2017 TOP 10!**

## The sources of logfiles and on which we'll be focusing

- Security Software - such as firewalls and anti-virus softwares
- Operating Systems
- Networking Equipment
- **Applications** <- That's the one I'll be writing today.


## Attack scenario chain where logging is crucial
1. Any type of brute-force attacks like **Credential Stuffing** - we could detect and prevent that during the attack.
1. When the attacker gets a foothold in e.g. internal network administrator panel and tries to learn about company's infrastructure - We could also detect this with logging
1. When the attacker performs Data Exfiltration, we could detect these big data transfers and also stop them
1. But **without any logging** and monitoring the defenders will learn about the **incident after the attacker requests a ransom!**

With logging, we could:
1. Detect the intrusion
1. Prevent worse from happening

## Insufficient Logging and Monitoring is...
- **Lack of Quantity** of logging events.

After that We've increased the Quantity we can focus more **about Quality** of these logs. For example add a timestamp.

## Other issues with Logging and Monitoring:
- Lack of Availability - When we can't access the logs
- No Alerting Thresholds - When even with the sufficient logs we don't get any alerts, and we can't have timely response.

## The process of Logging, Monitoring, Alerting.

To have the process completed, we need to have all these 3 elements (Infinity Stones!)
1. Logging
 - **What** event happened?
 - **When** the event happened?
 - **Where** the event happened?
 - **Who** performed the event?
2. Monitoring
- which you can achieve only with proper logging
- Providing timely detections
3. Alerting - Actively informing someone about the event that is peculiar.

Why not Log and Monitor everything?
- Legislation restricts the data we wish to log
- We might break Confidentiality 'rule'
- We might get Information Overload, which would be harder to analyze
- Cost of Processing logs would be much bigger.

## In which events are we interested?
1. Authentication events (**Successes** and **Failures**)
2. Authorization events (When the user tries to access a certain parts of the application - Focus on **Failures**)
3. Use of Privileges (When certain administrative function is invoked - **Successes** and **Failures**)
4. Application Specific Events **Decide whether it makes sense to log these**(Application errors, Startup and shutdown, Configuration changes, Application state information, Input and output validation failures)

## The place and timely of when to decide on what to log:
If We refer to SDLC life-cycle ideally we should consider what to log in
- Defining Requirements phase
- Designing phase

However, **enabling monitoring, tweaking it and configuring logging** is done in **Deployment phase**

## Improving the Quality of the Logged Data

### Where should they be stored?
- On a remote logging server like  **AWS CloudFormation**, **Cloud Logging in Google Cloud**
- Locally (**Which is not recommended**, However if it's inevitable then we should store in on a separate partition/disks and in some way encrypted with limited access (which would damage the Availability of the data - that's why It's not recommended))
### The Format/Contents of Log files:
 1. **What** event happened?
 - The type of event - what happened?
 2. **When** the event happened?
 - When the log entry was generated on the application side
 - When the log server received this log
All of that with **synchronized time sources!**
- Take time zones into consideration also. (Standard RFC 3339)
 3. **Where** the event happened?
 - Log source address - responsible for logging the entry
 - Originating address - Responsible for generating the event
 4. **Who** performed the event?
- Logged on user / Unique Identifier

**Log standards**:
- Syslog - Standard RFC 5424
- **Common Log Format**
- Use a company standard, when available - This would simplify and integrate with the existing logging functionality

### Syslog Severity Events
Many Log Standards also log severity as metadata - how serious is the event:
- 0 - Emergency: system is unusable
- 1 - Alert: Action must be taken immediately
- 2 - Critical: Critical conditions
- 3 - Error: error conditions
- 4 - Warning: warning conditions
- 5 - Notice: normal but significant condition
- 6 - Informational: informational messages
- 7 - Debug: debug-level messages

It's good to be consistent with these Severity Levels and not use different type of severity levels on a different application.

### Anonymizing the data
Often refers to replacing the UUID with the random value. To achieve this result we can:
- Encrypt the Data
- Pseudonymization

**Encryption** - is what it's normally stands for - converting personal data to encrypted format. This though requires the Encryption Key and managing these secrets also is troublesome. If we revoked the secret then all the backed up logs would become useless. Decryption of these logs will also affect the performance.

**Pseudonymization** - Replacing personal data with an artificial identifier (pseudonym), requires Link table that would link the pseudonym to original data

## Log Management

### Log File Life cycle
1. Generation
1. Transmission
1. Aggregation
1. Analyzing
1. Archiving

**Aggregation** - Means to:
- remove duplicate events (very rare in WebApps),
- Add structure,
- Remove Sensitive Fields if these weren't remove in previous stages.
- Add Input Validation, Encoding, White-Listing/Black-Listing - Because there's still an option for an attacker to perform **Log Poisoning**

**Analyzing**
- Determine what is normal - This takes time
- Anomaly detection
- Using Attack signatures
- Escalation, Response, Alerting

**Archiving**
- Moving data from hard storage to back up media.
- Preserving Decryption keys if it's needed
- Securing Log Data

### Response Strategies
- We could start from the existing security policies (NIST-800-61)
- Preparation -> identification -> containment


## Available Solutions
- [OWASP AppSensor](http://appsensor.org/)
- [OWASP ModSecurity (Open-Source WebApp firewall)](https://owasp.org/www-project-modsecurity-core-rule-set/)
- [ELK Stack (Elastic Search, Logstash, Kibana)](https://www.elastic.co/what-is/elk-stack)
- Security Information Event Management (SIEM) - To manage them

## Testing, Testing, Testing!
- Observe alerts
- Check quantity and quality of the logs
- Validate procedures (Check if these are false alerts)
- Recheck configuration.

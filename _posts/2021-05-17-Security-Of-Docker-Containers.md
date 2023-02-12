---
title: Security of Docker Containers
date: 2021-05-17 06:46:00 +0100
categories: [DevSecOps, Docker]
tags: [devsecops, docker, containers]
lang: en
---

## Intro

The use of Docker Containers has changed completely on how we deploy our applications, many though think that all the apps running in the container are completely isolated from the host system and at the same time secure by default (Partially there are, but thinking something is 100% secure is bad assumption.)On this blog-post I'll write about many mistakes in building Docker containers regarding their security.

## Not creating non-root user on our container.

We should always in Dockerfile create a non-root user and switch to it, after we perform all configuration in the Dockerfile requiring root user

Many of the base image containers already set up non-root user, we only need to write the line
```md
USER <created_username>
```
Somewhere at the end of our Dockerfile.


If you don't want to check it up, you can create it with:
```md
RUN groupadd -gid 1000 <username> \
&& useradd -uid 1000 -gid node --shell /bin/bash --create-home node
```

## Not building upon a distroless image

When we base our image on some Linux Distribution, it contains many Linux executables like bash shell. The attacker might use these commands to do a fancy privilege escalation technique.

To limit his actions and surface attack beforehand, we should build our app upon a distroless image. We do that with Multi-Stage building.

Here's Dockerfile examples:

### Golang App
```md
# Start by building the application.
FROM golang:1.13-buster as build

WORKDIR /go/src/app
ADD . /go/src/app

RUN go get -d -v ./...

RUN go build -o /go/bin/app

# Now copy it into our base image.
FROM gcr.io/distroless/base-debian10
COPY --from=build /go/bin/app /
CMD ["/app"]
```
### Java app
```md
FROM openjdk:11-jdk-slim AS build-env
ADD . /app/examples
WORKDIR /app
RUN javac examples/*.java
RUN jar cfe main.jar examples.HelloJava examples/*.class

FROM gcr.io/distroless/java:11
COPY --from=build-env /app /app
WORKDIR /app
CMD ["main.jar"]
```
### NodeJS app
```md
FROM node:10 AS build-env
ADD . /app
WORKDIR /app

RUN npm ci --only=production

FROM gcr.io/distroless/nodejs:10
COPY --from=build-env /app /app
WORKDIR /app
CMD ["hello.js"]
```
Now, lets remember, that **these distroless application do not have a shell!** Because of this, we need to specify the ```ENTRYPOINT [<app_name>]``` inside our Dockerfile. This way the app container won't default to ```CMD []``` statement (which uses bash) and we'll be able to interact with our application in a distroless environment.

## Looking to the Hosts System Hardening
The examples on what things to look for are:
- **The containers should be run in a virtualized environment**
- **Using Container Optimized OS** e.g. from Google.
- Hard Disk Encryption
- System updates
- Removing unnecessary packages
- Checking for open ports
- Enabling SELinux(Security Enchanced Linux), AppArmor
- Password policies to report any failed attempt of login
- Access Control

## Not using Container Security Image Scanner
For example:
- anchore
- clair
- OpenSCAP
- DockerBench
- within many Cloud providers

You should use them :P

## Using Rootless Docker 'Mode' badly.

By default, Docker daemon requires root privileges. Sometimes users don't want to provide every time a password when attempting to run a container [just like in this post](https://askubuntu.com/questions/477551/how-can-i-use-docker-without-sudo).

There are few things to consider here:
### Only trusted users should be allowed to control your Docker daemon

This is a bad practice, because an attacker might set up shared folders to your **root directory** ```/```
Then from the app/container context it could modify your host filesystem without any obstacles, e.g. create a new - more vulnerable container.

He could for example interact with Docker daemon socket in ```/var/run/docker.sock``` With that, the attacker would have a full control of the host machine.

## Don't configure a service blindly before getting to know it better

Misconfigurations are part of OWASP TOP 10, and these.
Some configuration you'd like to perform may seem complex, in that case the solution is simple, don't use complex configurations, because there could be a lot of potential places to make mistakes

Also, do your own research on dependencies

## Linux kernel capabilities.

By default, Docker runs containers with a restricted set of capabilities.

The capabilities itself simply provide implementation of The Least Privilege Principle.
For example, network service that binds to a port doesn't need root permissions, it only needs ```net_bind_service``` capability.

The secure-by-default implementation of Docker Capabilities is amazing, and It even makes it better than running ssh daemon, ```cron``` on a host machine.

But the default set of capabilities may still not provide full isolation of the app, therefore We can remove the capabilities from the allow list.

But **beware with adding new capabilities**, make sure the capability is 100% needed for the application to run properly.

## Running docker images with ```-privileged``` flag

**do not run docker images with ```-privileged``` flag!**

Instead, run it with ```–no-new-privileges```
## Not keeping our images up-to-date

This goes without saying, having updates also provide us with security patches of the applications, cmdline tools our container needs.

This though should not mean running always image with ```:latest``` tag, because it can bring on other issues with Availability. More on that on [derick's blog-post](https://derickbailey.com/2017/05/10/never-use-the-latest-image-from-docker-hub/)

## Not disabling inter-container communication.

It can be done with ```--icc=false```

By default it is enabled, meaning that all containers can talk with each other using ```docker0 bridged network```

If some containers need communication, they can be specified with
```--link=CONTAINER_NAME_or_ID:ALIAS```

## Not limiting DoS attacks.

Docker daemon assigns resources if needed automatically, which is a good thing, but in the DoS attack scenario, the Docker daemon will request too much RAM and CPU than a Host machine can provide, resulting in killing/restarting the process on Host by the kernel.


It can be done with limiting container's access to memory with the ```--memory=``` flag,
```--memory-swap```

Also, We can limit maximum number of restarts with

```--restart=on-failure:<number_of_restarts>```

flag
and maximum number of processes with

 ```--ulimit nproc=<number>```

## Not setting filesystem and volumes to read-only.

Can be done with
```--read-only``` flag when performing ```docker run```.

If it has to save something, we can use ```--tmpfs /tmp``` flag with /tmp directory.

## Not signing an image
If someone happens to get to the Host OS some way, **We should sign our container images** to prevent them from tampering with this image and modifying its behavior


## Other places to look for:
### Security of IMAGE REGISTRY
- Keeping them private
- Monitor the registry
- Also provide Security of Host System hosting IMAGE REGISTRY
### Not monitoring Network around containers

E.g. with Prometheus

### Security of Container Orchestration
More on that in a future blog-post

## References
The knowledge I provide here is not mine of course, the best resources that I found valuable were:
- [OWASP Docker Security Docs](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
- [DevOps Directive's awesome video!](https://www.youtube.com/watch?v=JE2PJbbpjsM)
- [Docker Security Docs](https://docs.docker.com/engine/security/)
## Summary
Docker containers are mostly secure-by-default, but We shouldn't underestimate the importance of Host system where they are being run, and also the importance of stripping these images even more like we would do with ```distroless``` concept.

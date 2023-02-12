---
title: Basics of Kubernetes (Without Hands-on Practice)
date: 2021-05-03 06:46:00 +0100
categories: [DevSecOps, Kubernetes]
tags: [devsecops, kubernetes, containers]
lang: en
---

# Intro:
## What I want you to know already:
- What are containers - I recommend this course -> [freeCodeCamp Docker](https://www.youtube.com/watch?v=fqMOX6JJhGo)
- How to create them in Dockerfiles, for that I recommend blackarch-zsh [blog-post](https://cloufish.github.io/blog/posts/creating-your-own-docker-pentesting-container/) where I explain how I created blackarch-zsh container.

## What is Kubernetes?
It's a way of orchestrating containers...
- It's made by Google
- While being still open-source project
- Stable and mature, even though one of its first version was back in 2015!
- It's shortened name is K8s - pronunciation of this is "Keights" (I know this may not be accurate pronunciation to interpret)

## Why we use it:
- To manage containers and microservices of course,
- But also it doesn't require developer to know what hardware he wants to use

## Explaining the Jargon:
- **monolith** - a massive application with all the code and elements bundled into one environment

![monolith](https://imgur.com/Om67W4j.png)
- Microservice App - when all these elements from monolith app are divided into little micro-services

![micro-service](https://imgur.com/oCOCjdL.png)
- **Orchestration** - refers into combining/organizing these micro-services to work together


The **Pros** of Microservices are:
- They can be scaled more independently
- Also developed, patched, updated independently
- It can be run anywhere

**Cons**:
- It can be complex!

# Kubernetes Architecture

## K8s cluster
Is a place where we store all of our **nodes** and **Master Control Plane**/**Kubernetes Mater**

On these nodes many **Pods** run

![cluster](https://imgur.com/ULHozud.png)

I know it does sound complicated, but hang on! As you read this blog-post, write these stuff down it'll become clear!


**Master Control Panel**:
- Monitors the cluster
- Makes the changes
- Schedules the work
- Reacts to event

**Node**:
- Where we run our Application



## Process of Packaging the application:

1. Refers to containerizing our app - Make it run as the container
2. Then we wrap that into a **Pod** (it's needed by Kubernetes!)
3. We also need to wrap it into **deployment** controller - If we want it to be scalable.

4. We perform all these steps in a **file**!
5. We then transfer this configuration file to the Kubernetes **Master**, and then it handles the situation by himself.

## Masters / Control Plane
- Brain of the cluster

**Multi-master Control Plane** - Concept stands for splitting the Control Plane into smaller ones which would be responsible for a **particular** failure domains - because sticking them on the same data-center/RACK is definitely **not recommended**

**How many Masters to have?** - 3 or 5, but definitely not an even number! (not 2 for example), because if one master fails, the remaining clusters must have a majority! If for example we have 2 masters and one of them fails, then the remaining one master will transform into **read-only mode.**

**Active-Passive Multi Master Model** - Where only one master is only actively making changes to the cluster, while other masters reflect/copy what the **leader** master did

Every Master runs on **a separate Linux environment**

Every master runs every **master-component**

**Hosted K8s Control Plane**
- in Cloud-Hosted Masters are hidden from you
- Cloud-Provider run your Control Plane **as a service**, and you can interact with it with API endpoint

>**WARNING** - Don't run user or business application on a Master

**kube-apiserver** — is a Front-end to the control plane
- Exposes the API
- Consumes JSON/Yaml

**Cluster store** - Where the config, state, apps running on master gets stored
- Persists cluster state and config
- Based on etcd
- With that Performance is critical
- You should have backups of these Cluster store and regularly test them

**Kube-controller-manager**
- Controller of other controllers like:
  1. Node controller
  1. Deployment controller
  1. Endpoints controller
- Watches loops
- Observes state and compare it with a desired one

**Kube-scheduler**:
- Watches API Server for new work tasks
- Assigns work to nodes
  - Affinity
  - Constraints
  - Taints
  - Resources
  - (Really complex stuff in general)

![Master](https://imgur.com/x6P62BZ.png)

## Nodes

**kubelet**:
- Main Kubernetes agent that runs on every node
- Registers node with cluster
- Watches API Server for work tasks (Pods)
- Executes Pods
- Reports back to Masters

**container-runtime**
- Pluggable: Container Runtime Interface (CRI)
  - Docker (getting deprecated),
  - containerd,
  - CRI-O,
  - Kata
- Low-level container intelligence

**kube-proxy**
- Networking component
- Pod IP addresses
- Lightweight Load-Balancing across all the Pods behind the service


![node](https://imgur.com/UCzziwk.png)

## Declarative way of configuring Kubernetes
- In configuration file, we only **declare** **what we want** the Kubernetes to 'look like'. In Declarative way we don't care on how to achieve everything, this is handled by the Kubernetes itself!

If we declared that...:
- We always want 3 instances of Front-End Pod running
But when one of these Pods are taken down, the desired state vs observed state is different! When that happens, Kubernetes brings up the new Pod or even a Node.

## Pods

It consists of containers (e.g. Docker ones).
For containers to be orchestrated - **They must be in a Pod!**

The more accurate **definition of Pods** is that they're shared execution environment for containers to run on
All containers in the same Pod share the same resources

It's mostly preferred though to run only one container in single Node, because usually there's no need of sharing resources

By the term of **scaling** we mean **increasing the number of Pods running**, not the number of containers in a single Pod

However, there is a use case of running multi-container Pods, such as:
- Stuff that shouldn't be done on the App directly, due to security reasons like **Decrypting, Encrypting** traffic over the network, and other networking stuff

Like we said earlier, **Pods do not offer Scalability** by default for them to be scalable, they must be wrapped into a **Deployment** controller, but Pods themselves provide nice meta-data for Kubernetes to work with!
## Networking with Kubernetes Services.

When new Pods are re-deployed (e.g. after it dies) or after scaling up, or even updating They got assigned a new IP Address, which can be **problematic from Networking** Administrator's perspective

When We want to make Networking in Kubernetes easier we can spin up **Service Object**. It sits in front of the Pods and provide them a stable IP and DNS names and it Load-Balances them.
It handles Networking partially for us.

### Labels
- Are the way of categorizing the Pods, we can put the labels like:
- Version of the App e.g 1.3
- prod/dev label
- back-end / front-end

We can then connect these Labels with our Networking Service Object,
Then send a declarative 'command'/request that 'I want all 1.3 Pods to be updated into 1.4 version and to label them as 1.4` - Ta-daa!
And also, If We want to **get rid of the 1.3 versions** to be managed, then we only need to do one thing... **Delete the label '1.3' from the Network Service Object**.
And then these Pods won't get **any** traffic!

There are other things to mention with Network Service Objects:
- Only sends traffic to the healthy Pods
- Can do both TCP and UDP
- Can send traffic to endpoints outside the cluster

## Deployment Controllers
- Watches API Server for new Deployments
- Compares observed state with desired state
- Enabled Self-Healing, running updates, version rollbacks

So we **declare a desired state** in config file, for example that 'We want 4 Pods/Replicas running'

Deployment Controller watches the observed state with desired state and fixes any abnormality to the desired state, as we discussed.

## Kubernetes API and API Server

Kubernetes run very different objects and combines them together like Deployment Controllers, Network Service Object, Pods

API is like catalog of features/services with.

The API helps us to better communicate with Kubernetes API Server and vice-versa

We use ```kubectl``` command line tool to communicate with the API Server and also to transfer Manifests (Config files) to the API K8s Server
a
## Summary of the K8s Architecture:
![Summary](https://imgur.com/GZL3J3t.png)

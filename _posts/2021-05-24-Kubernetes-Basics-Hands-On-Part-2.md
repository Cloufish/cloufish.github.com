---
title: Basics of Kubernetes Part 2
date: 2021-05-24 06:46:00 +0100
categories: [DevSecOps, Kubernetes]
tags: [devsecops, kubernetes, containers]
lang: en
---

## Connecting to the Pod/App.

We know from previous blog-post, that because Pods are disappearing, redeployed all the time we can't directly connect to them via IP - these are changing constantly.

We then discussed a topic of Networking **Service Object**

The Service Object from the Host/front-end perspective, acts as a static IP address, where from the Pod's/back-end's perspective it acts as a Load Balancer, assigning the resources and with that also IPs dynamically.

When we connect to the Pods through Service Object, we do that using **ClusterIP**, but also **DNS name** assigned to this IP.

The DNS names are then transmitted to the Front-end Pods/Host config files like ```/etc/resolv.conf```. Thus, every Pod knows how to resolve these addresses.
### Communication between Apps
To better illustrate resolving DNS names between Apps:
1. App2 Node wants to communicate with App1 Node
2. It then sends a query to **Cluster DNS** server and asks - "I want to communicate with app1.svc, give me app1.svc IP address"
3. The ClusterDNS resolves this DNS name to a IP address and returns it to the App2 Node
4. With having the IP address of the App1 (His Service Network Object), the App2 sends a request to the App1 Service Network Object with their query to the App.
5. The Network Objects then queries one of the Pods and returns the results back to the App2 Node

### Communication between App/Pods and Host system on the same network.
This doesn't involve ClusterDNS, instead, it uses a **NodePort**.


All the nodes running are having the same Open Port. So, the communication is as follows:
1. From the Host Machine, we send request to any Node on that Port
1. Then it is routed to the Pod's Service Network Object, and then to the one of the Pods behind the Service

#### Imperative way of doing it:
With command:
```kubectl expose pod <pod_name> --name=<chosen_service_name> --target-port=8080 --type=NodePort```

The port we specify above is **from the Pod's perspective**, If we want to see the port that is mapped to our host machine, we can check it with command:

```kubectl get svc```

And there you'll see your service with the service name you specified, with the mapped port.

![get-svc](https://imgur.com/CH1Fck2.png)

#### Declarative way of doing it:

Setting the ```type: NodePort``` in the ```spec:``` object

```yml
apiVersion: v1
kind: Service
metadata:
  name: ps-nodeport
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 31111
    protocol: TCP
  selector:
    app: web

```
```port: 80``` - refers to the port App listens on **inside the cluster**

```targetPort: 8080``` - refers the port App listens on **inside the container**

```targetPort: 31111``` - refers to the port Nodes listens on for our Host machine.

### Communication between App/Pods and Host system over the Internet.

It uses **Load Balancer**.

The Load Balancer integrates with Cloud Load Balancers

It doesn't require much configuration, only to specify ```type: LoadBalancer``` in the yaml file, withing ```spec:``` object

Well... it does require one thing - That the cloud implements Load Balancing, but most of the cloud provide it!

## Kubernetes Deployment

K8s Deployment Service provide us a way of interacting with the **replica sets**.

Replica sets stores a meta-data of desired state. It also watches this desired state constantly and at the same time It does scaling and self-healing.

So the Deployment Service is used mostly to update the meta-data to be stored in replica sets.
With it, We can also do rollback of the Deployment Manifest

When We declare in the Deployment Service Manifest that we want a smaller amount of replicas e.g. 3, but we had 6, then the 3 **replicas will become inactive, but not deleted**. This is to make the possibility of **rollbacks**

### Writing Deployment Controller Manifest

In Earlier blog-post when we were deploying Guestbook, we used this one:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
  labels:
    app.kubernetes.io/name: mongo
    app.kubernetes.io/component: backend
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: mongo
      app.kubernetes.io/component: backend
  replicas: 1
  template:
    metadata:
      labels:
        app.kubernetes.io/name: mongo
        app.kubernetes.io/component: backend
    spec:
      containers:
      - name: mongo
        image: mongo:4.2
        args:
          - --bind_ip
          - 0.0.0.0
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 27017

```
I don't think I'll be writing about the same thing again, sorry, it just seems unlogical to me.

But let's add, that we can update this Manifest file and apply it with command:
```kubectl apply -f deployment.yml```
And the replicas will update their data like desired state if necessary, without any drop of the services, so amazing!

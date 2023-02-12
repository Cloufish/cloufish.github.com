---
title: Basics of Kubernetes (Hands-On Experience)
date: 2021-05-10 06:46:00 +0100
categories: [DevSecOps, Kubernetes]
tags: [devsecops, kubernetes, containers]
lang: en
---

## kubectl
A command-line tool to access the K8s API Server

### Installation

[Here are the docs to guide you with installation](https://kubernetes.io/docs/tasks/tools/)

On my host machine, I only run
  ```sudo pacman -Syu kubectl```

### Installing Kubernetes itself
We have 2 choices here
- Docker Desktop - Easier one, however, **only Windows and Mac are suppoerted** :/
- minicube - More difficult than Docker Desktop, but I think It's better to struggle a bit but get more accurate, command line experience

If you want to use **Docker-Desktop** because you're on Mac or Windows, then to install it head over to [HERE](https://www.docker.com/products/docker-desktop) and download .exe file

**minicube** - Head over [HERE](https://minikube.sigs.k8s.io/docs/start/) and proceed from their documentation

## Managing Pods
### Process Overview:
1. Build container image out of the app,
2. Store it in the container registry (e.g. Docker Hub)
3. Create Manifest file in ```.yml``` format that would define the Pod's structure
4. Send it to the K8s API Server

We'll focus on the 3 and 4 step, because we already have containerized apps - by it, I mean even the simplest ones, like ```mongodb``` container — that is also a container app!

This → [repo](https://github.com/kubernetes/examples) contains a lot of more examples to work on with Kubernetes

I'll be choosing ```Guestbook```. And because I'm also a learner just like you, I'll do everything as described in Kubernetes own [Tutorial/Guide](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/) from now on.


## Guestbook

### Starting minikube cluster
With ```minikube start```

Then, if we run ```kubectl get po -A```
```
NAMESPACE              NAME                                        READY   STATUS    RESTARTS   AGE
kube-system            coredns-74ff55c5b-87864                     1/1     Running   1          23d
kube-system            etcd-minikube                               1/1     Running   1          23d
kube-system            kube-apiserver-minikube                     1/1     Running   1          23d
kube-system            kube-controller-manager-minikube            1/1     Running   1          23d
kube-system            kube-proxy-bzqzx                            1/1     Running   1          23d
kube-system            kube-scheduler-minikube                     1/1     Running   1          23d
kube-system            storage-provisioner                         1/1     Running   2          23d
kubernetes-dashboard   dashboard-metrics-scraper-f6647bd8c-6tz59   1/1     Running   0          6m18s
kubernetes-dashboard   kubernetes-dashboard-968bcb79-2226c         1/1     Running   0          6m18s

```

We'll see that there are various Kubernetes objects running like the ```etcd```, ```kube-apiserver-minikube```, ```storage-provisioner```.

Many of them we discussed in previous blog-post related to Kubernetes.

So as discussed in the kubernetes tutorial We need to create a Manifest of the **MongoDB Deployment Controller**

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
**Let's analyze this file more in depth.** I think It'll help us write Manifests by ourselves.

>1> ```apiVersion: apps/v1``` - That's easy to deduct, the API we use is from the **apps group**, and we use its v1

>2>```kind: Deployment``` - This though is not so intuitive in the first glance, but It specifies the **resource type**

With command ```kubectl explain --api-version=apps/v1 deployment``` We can get more info about this resource type

```yml
metadata:
  name: mongo
  labels:
    app.kubernetes.io/name: mongo
    app.kubernetes.io/component: backend
```
We see that be assign metadata to this mongo service more specifically **labels** which are the same as **tags** in other concepts,

```yml
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
The spec section defines what state we desire for the object and in this scenario there are lots of them, let's break it down again.

```yml
selector:
    matchLabels:
      app.kubernetes.io/name: mongo
      app.kubernetes.io/component: backend
```
We specify which Apps/Pods we want it to affect, here we see that we only want these apps which have ```mongo``` and ```backend``` labels.

```replicas: 1``` - Simple, we want 1 copy of the deployment controller

```yml
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
But this part is important, because **the replicas are made according to the template**, and it basically tells it to run Kubernetes Pods with defined labels, and use containers with specified image, hardware resources and open port

Why is ```spec``` object used two times? **The first time we used it to label our Deployment Component**, then the use case is for a ```template:``` for replicas. You will see that with the **Service component** we only use it once

The ```spec``` object format is not so consistent across other K8s objects, as we read the [API docs - kubernetes-objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)

>The precise format of the object ```spec``` is different for every Kubernetes object, and contains nested fields specific to that object. The Kubernetes API Reference can help you find the ```spec``` format for all the objects you can create using Kubernetes. For example, the ```spec``` format for a Pod can be found in PodSpec v1 core, and the spec format for a Deployment can be found in DeploymentSpec v1 apps.

**Okay, back to deploying our app!!!**

Write the contents to the ```deployment.yml```,
Then execute:

```kubectl apply -f mongo-deployment.yaml```

We'll get the output:

```
deployment.apps/mongo created
```
Then to get the Pod We've created run
```kubectl get pods```
```
NAME                     READY   STATUS    RESTARTS   AGE
mongo-75f59d57f4-8kjp2   1/1     Running   0          93s
```
So our Deployment Controller is running!

Now Let's create the **MongoDB Service**, with very similar steps:
1. Copying this yml content:
```yml
apiVersion: v1
kind: Service
metadata:
  name: mongo
  labels:
    app.kubernetes.io/name: mongo
    app.kubernetes.io/component: backend
spec:
  ports:
  - port: 27017
    targetPort: 27017
  selector:
    app.kubernetes.io/name: mongo
    app.kubernetes.io/component: backend

```
into mongo-service.yml file inside your App directory

> (Here as you can see we use only ```spec```.)

2. Apply it with
```kubectl apply -f mongo-service-yml```

And We'll **do the same** with **Guestbook Frontend Deployment** and **Guestbook Frontend Service** just like the [tutorial](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/) says

And verify if replicas were made with:

```kubectl get pods -l app.kubernetes.io/name=guestbook -l app.kubernetes.io/component=frontend```

And If you've done everything the docs specified, your WebApp will be running

![guesbook-frontend](https://imgur.com/LZqCIkl.png)

>I know that there were little to no work from my part, looks like these docs have everything explained, which is amazing!

### Conclusions with Guestbook
- Writing Manifests in ```.yml``` format is essential, that's basically the main part of configuring the services in K8s. The best way to learn it is of course by writing them and analyzing more complex ones, for different ```kind:``` objects.


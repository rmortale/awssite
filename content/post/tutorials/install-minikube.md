---
title: Install minikube
date: 2020-06-20T20:20:16+01:00
draft: false
author: Antonio
image: images/kubernetes.png
description: How to install minikube
categories: 
  - minikube
  - Kubernetes
---

In this tutorial we install [minikube](https://minikube.sigs.k8s.io/docs/). A single node Kubernetes Cluster which runs on our Laptop. Usefull for developing and testing.

#### Prerequisites
* [Docker installed](https://docs.docker.com/get-docker/)

#### Download and install minikube
I am using Linux for this Tutorial but minikube also works on MacOS and Windows. Download minikube for your Operating System from [here](https://github.com/kubernetes/minikube/releases). I am using version 1.11.0. You rename the binary downloaded to `minikube` and put it on your path.

#### Start the cluster
I am using the Docker driver for this Tutorial. I start minikube with this command:

    minikube start --driver=docker

After a minute or so the cluster should be up and running. If minikube fails to start, see the [drivers](https://minikube.sigs.k8s.io/docs/drivers/) page for help setting up a compatible container or virtual-machine manager.

     minikube v1.11.0 on Ubuntu 20.04
    ✨  Using the docker driver based on existing profile
    👍  Starting control plane node minikube in cluster minikube
    🔄  Restarting existing docker container for "minikube" ...
    🐳  Preparing Kubernetes v1.18.3 on Docker 19.03.2 ...
        ▪ kubeadm.pod-network-cidr=10.244.0.0/16
    🔎  Verifying Kubernetes components...
    🌟  Enabled addons: dashboard, default-storageclass, storage-provisioner
    🏄  Done! kubectl is now configured to use "minikube"


#### Test minikube
Try the command below and you should get all pods running in the `kube-system` namespace.

    minikube kubectl -- get pods --namespace kube-system

    NAME                               READY   STATUS    RESTARTS   AGE
    coredns-66bff467f8-4tvnj           1/1     Running   5          6d2h
    coredns-66bff467f8-nqg27           1/1     Running   5          6d2h
    etcd-minikube                      1/1     Running   4          3d1h
    kube-apiserver-minikube            1/1     Running   4          3d1h
    kube-controller-manager-minikube   1/1     Running   5          6d2h
    kube-proxy-z5n74                   1/1     Running   5          6d2h
    kube-scheduler-minikube            1/1     Running   5          6d2h
    storage-provisioner                1/1     Running   8          6d2h


Start the Kubernetes dashboard with the command `minikube dashboard`. The dashboard should open in your browser.

#### Stop the cluster
If you want to stop minikube enter `minikube stop`.

#### Summary
In this tutorial we installed and started minikube.


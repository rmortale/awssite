---
title: "Create a Kubernetes cluster on Azure"
date: 2020-07-11T09:11:59+01:00
draft: false
author: Antonio
image: images/aks.png
description: Create a Kubernetes cluster on Azure
categories: 
  - Kubernetes
  - AKS
  - Azure
---

In this post we will use the CLI to create a Kubernetes Cluster on Azure (AKS). You can follow the video or continue reading.

{{< youtube lcf-TiPobU4 >}}

### Prerequisites
* [Azure account](https://azure.microsoft.com/en-us/)
* [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) installed and configured
* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) installed

### Create a Kubernetes Cluster
The Azure resources we create are not free. Be sure to clean them up at the end! To create the cluster we have to create a resource group first. The following command creates a resource group named `rg-cluster` in the `eastus` region. Choose the [region](https://azure.microsoft.com/en-us/global-infrastructure/regions/) nearest to you.
    
    az group create --name rg-cluster --location eastus

Now create the AKS cluster named `aks-demo` with one node in our new resource group

    az aks create --resource-group rg-cluster --name aks-demo --node-count 1

After a few minutes, the command completes and returns JSON-formatted information about the cluster.



### Connect to the cluster
First get the cluster credentials.

    az aks get-credentials --resource-group rg-cluster --name aks-demo

This will update your Kubernetes configuration. Now we can connect to our cluster.

    kubectl get nodes
    
    NAME                                STATUS   ROLES   AGE   VERSION
    aks-nodepool1-33611276-vmss000000   Ready    agent   12m   v1.16.10


    kubectl get pods --all-namespaces
    
    NAMESPACE     NAME                                         READY   STATUS    RESTARTS   AGE
    kube-system   coredns-544d979687-qq2mr                     1/1     Running   0          16m
    kube-system   coredns-544d979687-rs2f4                     1/1     Running   0          13m
    kube-system   coredns-autoscaler-98c475c7d-rv76g           1/1     Running   0          16m
    kube-system   dashboard-metrics-scraper-5f44bbb8b5-gxbxf   1/1     Running   0          16m
    kube-system   kube-proxy-xd2lc                             1/1     Running   0          14m
    kube-system   kubernetes-dashboard-785654f667-74k8v        1/1     Running   0          16m
    kube-system   metrics-server-85c57978c6-5f8db              1/1     Running   0          16m
    kube-system   tunnelfront-57cdbb4775-47lmb                 1/1     Running   0          16m

It works! The Kubernetes cluster is up and running.

### Delete
To delete all resources created use:

    az group delete --name rg-cluster

### Summary
In this post we created a Kubernetes cluster on Azure.


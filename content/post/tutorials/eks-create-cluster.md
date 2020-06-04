---
title: "Install Kubernetes on AWS with one command"
date: 2020-06-01T09:11:59+01:00
draft: false
author: Antonio
image: images/eks-aws.png
description: Install Kubernetes on AWS with eksctl
categories: 
  - Kubernetes
  - EKS
---

In this post we will use [eksctl](https://eksctl.io) to create a Kubernetes Cluster on AWS (EKS). You can follow the video or continue reading.

[![AWS](https://static.shareasale.com/image/43514/468X6010.jpg)](https://shareasale.com/r.cfm?b=1373702&amp;u=2310472&amp;m=43514&amp;urllink=&amp;afftrack=)

{{< youtube 6cYgKoqqZ_I >}}

### Prerequisites
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv1.html) installed and configured
* A specific version of [kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html) which works with EKS

### Install eksctrl
Download the latest release of [eksctrl](https://github.com/weaveworks/eksctl/releases) for your OS. Extract the binary and put it on your path.

### Create a Kubernetes Cluster
The AWS resources we create are not free. Be sure to clean them up at the end! To create the cluster enter
    
    eksctl create cluster

A EKS cluster will be created with default parameters

* auto-generated name
* 2x m5.large worker nodes using official AWS EKS AMI
* dedicated VPC

After some minutes the cluster is up and running. Example output

    $ eksctl create cluster
    [ℹ]  eksctl version 0.19.0
    [ℹ]  using region eu-west-1
    [ℹ]  setting availability zones to [eu-west-1b eu-west-1a eu-west-1c]
    [ℹ]  subnets for eu-west-1b - public:192.168.0.0/19 private:192.168.96.0/19
    [ℹ]  subnets for eu-west-1a - public:192.168.32.0/19 private:192.168.128.0/19
    [ℹ]  subnets for eu-west-1c - public:192.168.64.0/19 private:192.168.160.0/19
    [ℹ]  nodegroup "ng-96ad4497" will use "ami-023736532608ff45e" [AmazonLinux2/1.15]
    [ℹ]  using Kubernetes version 1.15
    [ℹ]  creating EKS cluster "attractive-mongoose-1591006576" in "eu-west-1" region with un-managed nodes
    [ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial nodegroup
    [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=eu-west-1 --cluster=attractive-mongoose-1591006576'
    [ℹ]  CloudWatch logging will not be enabled for cluster "attractive-mongoose-1591006576" in "eu-west-1"
    [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --region=eu-west-1 --cluster=attractive-mongoose-1591006576'
    [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "attractive-mongoose-1591006576" in "eu-west-1"
    [ℹ]  2 sequential tasks: { create cluster control plane "attractive-mongoose-1591006576", create nodegroup "ng-96ad4497" }
    [ℹ]  building cluster stack "eksctl-attractive-mongoose-1591006576-cluster"
    [ℹ]  deploying stack "eksctl-attractive-mongoose-1591006576-cluster"
    [ℹ]  building nodegroup stack "eksctl-attractive-mongoose-1591006576-nodegroup-ng-96ad4497"
    [ℹ]  --nodes-min=2 was set automatically for nodegroup ng-96ad4497
    [ℹ]  --nodes-max=2 was set automatically for nodegroup ng-96ad4497
    [ℹ]  deploying stack "eksctl-attractive-mongoose-1591006576-nodegroup-ng-96ad4497"
    [ℹ]  waiting for the control plane availability...
    [✔]  saved kubeconfig as "/home/nino/.kube/config"
    [ℹ]  no tasks
    [✔]  all EKS cluster resources for "attractive-mongoose-1591006576" have been created
    [ℹ]  adding identity "arn:aws:iam::601912882130:role/eksctl-attractive-mongoose-159100-NodeInstanceRole-UBEZMGNEQG1G" to auth ConfigMap
    [ℹ]  nodegroup "ng-96ad4497" has 0 node(s)
    [ℹ]  waiting for at least 2 node(s) to become ready in "ng-96ad4497"
    [ℹ]  nodegroup "ng-96ad4497" has 2 node(s)
    [ℹ]  node "ip-192-168-37-243.eu-west-1.compute.internal" is ready
    [ℹ]  node "ip-192-168-70-171.eu-west-1.compute.internal" is ready
    [ℹ]  kubectl command should work with "/home/nino/.kube/config", try 'kubectl get nodes'
    [✔]  EKS cluster "attractive-mongoose-1591006576" in "eu-west-1" region is ready

Check the cluster

    kubectl get nodes
    
    NAME                                           STATUS   ROLES    AGE     VERSION
    ip-192-168-37-243.eu-west-1.compute.internal   Ready    <none>   5m48s   v1.15.11-eks-af3caf
    ip-192-168-70-171.eu-west-1.compute.internal   Ready    <none>   5m41s   v1.15.11-eks-af3caf

    kubectl get pods --all-namespaces
    
    NAMESPACE     NAME                       READY   STATUS    RESTARTS   AGE
    kube-system   aws-node-tvj6r             1/1     Running   0          8m36s
    kube-system   aws-node-zvcfk             1/1     Running   0          8m29s
    kube-system   coredns-6658f9f447-nrzvz   1/1     Running   0          14m
    kube-system   coredns-6658f9f447-smq8j   1/1     Running   0          14m
    kube-system   kube-proxy-4mdgv           1/1     Running   0          8m29s
    kube-system   kube-proxy-rmb7p           1/1     Running   0          8m36s

### Delete
To delete all resources created use:

    eksctl delete cluster --name=auto-generated name from above

### Summary
In this post we created an EKS cluster with one command.

{{< figure src="https://bluehost-cdn.com/media/partner/images/antoniodol/488x160/488x160BW.png" target="_blank" link="https://www.bluehost.com/track/antoniodol/blue1" >}}

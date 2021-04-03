---
title: Create a Kubernetes Cluster on GKE
date: 2021-04-03T09:11:59+01:00
draft: false
author: Antonio
image: images/gke.png
description: Create a Kubernetes Cluster on Google Kubernetes Engine (GKE)
categories: 
  - Kubernetes
  - GKE
---

In this post we will create a Cluster on Google Kubernetes Engine (GKE). When you like to test something on a Kubernetes Cluster one possibility is to create this cluster on GKE using preemptible VMs. Preemptible VMs are Compute Engine VM instances that last a maximum of 24 hours in general, and provide no availability guarantees. Preemptible VMs are priced lower than standard Compute Engine VMs and offer the same machine types and options. Let's start.

### Prerequisites
* Google Cloud Account
* [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) installed and configured
* [kubectl](https://kubernetes.io/docs/tasks/tools/) installed

### Create a Kubernetes Cluster
The resources we create are not free. Be sure to clean them up at the end! To create the cluster enter
    
    gcloud container clusters create tutorial-cluster --num-nodes=1 --preemptible

A GKE cluster named `tutorial-cluster` will be created with one preemptible node in your default region and project.

After some minutes the cluster is up and running. Example output:

    ...
    Creating cluster tutorial-cluster in europe-west6-a... Cluster is being health-checked (master is healthy)...done.     
    Created [https://container.googleapis.com/v1/projects/kubedev-304717/zones/europe-west6-a/clusters/tutorial-cluster].
    To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/europe-west6-a/tutorial-cluster?project=kubedev-304717
    kubeconfig entry generated for tutorial-cluster.
    NAME              LOCATION        MASTER_VERSION   MASTER_IP      MACHINE_TYPE  NODE_VERSION     NUM_NODES  STATUS
    tutorial-cluster  europe-west6-a  1.18.16-gke.302  34.65.132.163  e2-medium     1.18.16-gke.302  1          RUNNING

Kubeconfig is already updated with the new cluster information. To check the cluster try:

    kubectl get nodes
    
    NAME                                              STATUS   ROLES    AGE     VERSION
    gke-tutorial-cluster-default-pool-b3829e23-8lx4   Ready    <none>   2m36s   v1.18.16-gke.302

    kubectl get pods --all-namespaces

    NAMESPACE     NAME                                                         READY   STATUS    RESTARTS   AGE
    kube-system   event-exporter-gke-564fb97f9-5hbrs                           2/2     Running   0          3m42s
    kube-system   fluentbit-gke-2947p                                          2/2     Running   0          3m31s
    kube-system   gke-metrics-agent-d6kv5                                      1/1     Running   1          3m31s
    kube-system   kube-dns-57fcf698d8-vpqlg                                    4/4     Running   0          3m37s
    kube-system   kube-dns-autoscaler-7f89fb6b79-krnjz                         1/1     Running   0          3m36s
    kube-system   kube-proxy-gke-tutorial-cluster-default-pool-b3829e23-8lx4   1/1     Running   0          2m23s
    kube-system   l7-default-backend-7fd66b8b88-9cfdd                          1/1     Running   0          3m42s
    kube-system   metrics-server-v0.3.6-7b5cdbcbb8-8qxc4                       2/2     Running   0          2m58s
    kube-system   pdcsi-node-sp9np                                             2/2     Running   0          3m30s
    kube-system   stackdriver-metadata-agent-cluster-level-78df8f745b-jcf4n    2/2     Running   1          2m39s

Success! Our new GKE cluster is up and running.

### Delete
When you are done with your testing then delete the cluster with:

    gcloud container clusters delete tutorial-cluster

### Summary
In this post we created an GKE cluster, tested the connection and deleted it.


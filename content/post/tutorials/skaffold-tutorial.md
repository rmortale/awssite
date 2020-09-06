---
title: "Continuous development for Kubernetes"
date: 2020-09-06T16:11:59+01:00
draft: false
author: Antonio
image: images/kubernetes.png
description: Used skaffold to continuously deploy changes to our local Kubernetes cluster
categories: 
  - Kubernetes
  - Skaffold
---

In this post we will use skaffold. Skaffold is a command line tool that facilitates continuous development for Kubernetes applications. You can iterate on your application source code locally then deploy to local or remote Kubernetes clusters. Skaffold handles the workflow for building, pushing and deploying your application.

[![Certified Kubernetes Administrator](https://static.shareasale.com/image/43514/Certified_Kubernetes_Administrator.jpg)](https://shareasale.com/r.cfm?b=1543562&amp;u=2310472&amp;m=43514&amp;urllink=&amp;afftrack=)

### Prerequisites
* [Docker](https://docs.docker.com/get-docker/) installed
* [Local Kubernetes](https://docs.docker.com/get-docker/) installed (Docker Desktop or minikube..)
* [Skaffold CLI](https://skaffold.dev/docs/install/) installed

### Get the example code
Clone the [Skaffold Git repository](https://github.com/GoogleContainerTools/skaffold.git) to your computer. Open a terminal window and change to the getting-started example.
    
    Directory of C:\Users\Antonio\develop\k8s\skaffold\examples\getting-started

    05/09/2020  12:32    <DIR>          .
    05/09/2020  12:32    <DIR>          ..
    05/09/2020  12:32               323 Dockerfile
    05/09/2020  12:32               141 k8s-pod.yaml
    05/09/2020  23:35               142 main.go
    05/09/2020  12:32               302 README.md
    05/09/2020  12:32               147 skaffold.yaml
                5 File(s)          1.055 bytes
                2 Dir(s)  72.805.728.256 bytes free



### Run skaffold in development mode
Start Docker and your local Kubernetes cluster. Enter `skaffold dev` in the getting-started directory.

    C:\Users\Antonio\develop\k8s\skaffold\examples\getting-started>skaffold dev
    Listing files to watch...
    - skaffold-example
    Generating tags...
    - skaffold-example -> skaffold-example:v1.14.0
    Checking cache...
    - skaffold-example: Found Locally
    Tags used in deployment:
    - skaffold-example -> skaffold-example:fb9ce4c9b736121f89e609980d95e983be03972dd08d3ab576ed92f230b228cb
    Starting deploy...
    - pod/getting-started created
    Waiting for deployments to stabilize...
    Deployments stabilized in 11.0015ms
    Press Ctrl+C to exit
    Watching for changes...
    [getting-started] Hello world!
    [getting-started] Hello world!
    [getting-started] Hello world!
    
The code gets compiled and the docker image built. The image is then deployed to the local Kubernetes cluster. The pod output is shown in the terminal. Now you can edit the code and after saving the changes the pod will be rebuilt and redeployed in seconds.

### Delete
Pressing `Ctrl-C` in the terminal window will undeploy the pod and exit skaffold.

### Summary
In this post we used skaffold to continuously deploy changes to our local Kubernetes cluster.

{{< figure src="https://bluehost-cdn.com/media/partner/images/antoniodol/488x160/488x160BW.png" target="_blank" link="https://www.bluehost.com/track/antoniodol/blue1" >}}

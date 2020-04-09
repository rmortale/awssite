---
title: Deploy your Docker app on AWS ECS with one command
date: 2020-04-05T10:20:16+01:00
draft: false
author: Antonio
image: images/image3721.png
description: How to deploy a Docker app on AWS ECS with one command
categories: 
  - Fargate
  - ECS
---

In this tutorial I will deploy a simple Java Rest application to AWS ECS. I will use the new command line tool [ecs-cli-v2](https://github.com/aws/amazon-ecs-cli-v2). You can follow the video or continue reading.

[![AWS](https://static.shareasale.com/image/43514/468X6010.jpg)](https://shareasale.com/r.cfm?b=1373702&amp;u=2310472&amp;m=43514&amp;urllink=&amp;afftrack=)

{{< youtube LcnsDeBbcKw >}}

### Prerequisites
* Java JDK 1.8 installed
* Maven 3.x
* Git client
* Docker
* AWS Account
* AWS CLI configured


### Installation of ecs-cli-v2
Download and unpack the [ecs cli](https://github.com/aws/amazon-ecs-cli-v2/releases) (v0.0.7 as of this writing). Put it on the path. Open a command line and test if the tool works:

    ecs-preview  --version
    ecs-preview version: v0.0.7


### Installation of the tutorial application
Clone the tutorial application and change to the created directory. Checkout the ecs-preview-tut branch.
    
    git clone https://github.com/rmortale/javalin-rest.git
    cd javalin-rest
    git checkout ecs-preview-tut

This is a simple Java REST Service:

    package ch.dulce;

    import io.javalin.Javalin;

    public class HelloWorld {
      public static void main(String[] args) {
        Javalin app = Javalin.create().start(7000);
        app.get("/", ctx -> ctx.result("Hello World"));
      }
    }

To build and test it localy use:

    docker build -t javalinrest .
    docker run -it -p 7000:7000 javalinrest
    curl localhost:7000
    Hello World

### Deploy the application to AWS ECS
The following steps create multiple resources in AWS that generate costs. Please follow the steps at the end to delete all resources.

    ecs-preview init --project javalin     \
        --app api                          \
        --app-type 'Load Balanced Web App' \
        --dockerfile './Dockerfile'        \
        --port 7000                        \
        --deploy

This will create a VPC, Application Load Balancer, an Amazon ECS Service with the sample app running on AWS Fargate. At the end you'll get a URL for the sample app running!

    Deployed api, you can access it at http://javal-Publi-DVHYFO5SU3DF-175675924.eu-west-1.elb.amazonaws.com

    curl http://javal-Publi-DVHYFO5SU3DF-175675924.eu-west-1.elb.amazonaws.com
    Hello World

Our service is up and running.

### Cleaning Up
As mentioned before, the AWS resources we created are not free. So to clean them up, we can run the following:

    ecs-preview project delete

### Summary
In this post i demonstrated the new ecs-cli-v2. With one command we created a Fargate Cluster and deployed our sample app to it.


---
title: AWS Fargate service with fargatecli
date: 2020-03-09T10:20:16+01:00
draft: false
author: Antonio
image: images/fargate2.png
description: How to create a service on AWS Fargate with fargatecli
categories: 
  - Fargate
  - ECS
---

In this tutorial I will distribute a simple Java Rest application to AWS Fargate. I will use [fargatecli](https://github.com/awslabs/fargatecli). You can follow the video or continue reading.

[![AWS](https://static.shareasale.com/image/43514/468X6010.jpg)](https://shareasale.com/r.cfm?b=1373702&amp;u=2310472&amp;m=43514&amp;urllink=&amp;afftrack=)

{{< youtube XsQPhQwZh_Y >}}

### Prerequisites
* Java JDK 1.8 installed
* Maven 3.x
* Git client
* Docker
* AWS Account
* AWS CLI configured


### Installation of fargatecli
Download and unpack the latest fargatecli [release](https://github.com/awslabs/fargatecli/releases) (0.3.2 as of this writing). Adjust the path so that fargatecli can be called from everywhere. Open a command line and test if fargatecli works:

    fargate --version
    fargate version 0.3.2

### Installation of the tutorial application
Clone the tutorial application and change to the created directory:
    
    git clone https://github.com/rmortale/javalin-rest.git

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

### Deploy the application in Fargate
The following steps create multiple resources in AWS that generate costs. Please follow the steps at the end to delete all resources.

### Creating a Security Group
According to the documentation, the Security Group should be created automatically. Since that didn't work for me, we create a group using the aws-cli.

    aws2 ec2 create-security-group --group-name fargate-cli-sg --description "Group for fargatecli tutorial"
    {
        "GroupId": "sg-043ab03ec99b60369"
    }

For our application we have to open port 7000. Use the GroupId returned from the previous command.

    aws2 ec2 authorize-security-group-ingress --group-id sg-043ab03ec99b60369 --protocol tcp --port 7000 --cidr 0.0.0.0/0

Now we can distribute the application (Use the region where you want to create the resources and where the security group above is).

    fargate service create --port http:7000 --region eu-west-1 --security-group-id sg-043ab03ec99b60369 javalin-app
    ...
    [i] Created service javalin-app

The above command creates the following resources:
* Rebuilds our Docker container using the Dockerfile in the current directory.
* Creates an ECR (Elastic Container Registry) and pushes the container to it.
* Creates a Fargate cluster named "fargate" if one doesn’t already exist.
* Creates a "javalin-app" service in this cluster
* Creates a task definition with the protocol and port mapping described above. The task definition also specifies which container registry and tag should be used to pull our container image.
* Starts a task based on that task definition.

### List the running Service

    fargate service list --region eu-west-1

Show more information about the Service:

    fargate service info javalin-app --region eu-west-1
    34.244.147.151

The above command also shows the IP of the running task. To test it we use curl:

    curl 34.244.147.151:7000
    Hello World

Our service is deployed and responding.

### Cleaning Up
As mentioned before, the AWS resources we created are not free. So to clean them up, we can run the following:

    fargate service scale --region eu-west-1 javalin-app 0
    Scaled service javalin-app to 0

    fargate service destroy --region eu-west-1 javalin-app
    Destroyed service javalin-app

### Summary
In this post i demonstrated the fargatecli. With a few commands we created a Fargate Cluster and a Service with one Task. With fargatecli it is also possible to manage Load Balancers, Certificates and more. More about that in another post.

{{< figure src="https://bluehost-cdn.com/media/partner/images/antoniodol/488x160/488x160BW.png" target="_blank" link="https://www.bluehost.com/track/antoniodol/blue1" >}}

---
title: Deploy an AWS Fargate service with Github Actions
date: 2020-03-17T10:20:16+01:00
draft: true
author: Antonio
image: images/fargate2.png
description: How to deploy an AWS Fargate service with Github Actions
categories: 
  - Fargate
  - ECS
---

In this tutorial I will deploy a simple Java Rest application to AWS Fargate. I will use Github Actions to build and deploy the application.


### Prerequisites
* Java JDK 1.8 installed
* Maven 3.x
* Git client
* Github account
* Docker
* AWS Account
* AWS CLI configured

### Create the tutorial application
Login to your Github account and create a new repository. Clone the new empty repository to your machine. Download the [Sample application](https://github.com/rmortale/javalin-rest/tree/tutorial) as ZIP File. Unzip it and copy all Files to your new repository.

    .
    ├── .github
    │   └── workflows
    │       └── aws.yml
    ├── .gitignore
    ├── Dockerfile
    ├── LICENSE
    ├── README.md
    ├── create-service.bat
    ├── httpd-td.json
    ├── pom.xml
    ├── setup.bat
    ├── src
    │   └── main
    │       └── java
    │           └── ch
    │               └── dulce
    │                   └── HelloWorld.java
    └── taskdef.json
    
##### Create the ECR repository
    aws ecr create-repository --repository-name javalin-repo --region eu-west-1

Ensure that you have an ecsTaskExecutionRole IAM role in your account. You can follow the [Amazon ECS Task Execution IAM Role guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html) to create the role.

##### Register a task definition with:
    aws ecs register-task-definition --region eu-west-1 --cli-input-json file://httpd-td.json

##### Create an ECS cluster
Create the ECS cluster in the default VPC with this command:

    aws ecs create-cluster --region eu-west-1 --cluster-name fargate

##### Create a security group

##### Create the Fargate service
    aws ecs create-service --region eu-west-1 --service-name javalin-service --task-definition sample-fargate --desired-count 1 --launch-type "FARGATE" --network-configuration "awsvpcConfiguration={subnets=[subnet-fcc5779b,subnet-f7a318be],securityGroups=[sg-043ab03ec99b60369],assignPublicIp=ENABLED}" --cluster fargate

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


[![AWS](https://static.shareasale.com/image/43514/468X6010.jpg)](https://shareasale.com/r.cfm?b=1373702&amp;u=2310472&amp;m=43514&amp;urllink=&amp;afftrack=)


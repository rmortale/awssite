---
title: Deploy an AWS Fargate service with Github Actions
date: 2020-03-17T10:20:16+01:00
draft: false
author: Antonio
image: images/fargate2.png
description: How to deploy an AWS Fargate service with Github Actions
categories: 
  - Fargate
  - ECS
---

In this tutorial we will deploy a simple Java Rest application to AWS Fargate. We will use Github Actions to build and deploy the application. You can follow the video or continue reading.

[![AWS](https://static.shareasale.com/image/43514/300X2503_00.jpg)](https://shareasale.com/r.cfm?b=1551034&amp;u=2310472&amp;m=43514&amp;urllink=&amp;afftrack=)

{{< youtube FxvOpwOkCoQ >}}

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

##### Register a task definition
    aws ecs register-task-definition --region eu-west-1 --cli-input-json file://httpd-td.json

##### Create an ECS cluster
Create the ECS cluster in the default VPC with this command:

    aws ecs create-cluster --region eu-west-1 --cluster-name fargate

##### Create a security group
For the Fargate service we need a security group.

    aws ec2 create-security-group --group-name fargate-sg --description "Group for fargate tutorial"
    {
        "GroupId": "sg-043ab03ec99b60369"
    }

For our application we have to open port 7000. Use the security GroupId returned from the previous command.

    aws ec2 authorize-security-group-ingress --group-id sg-043ab03ec99b60369 --protocol tcp --port 7000 --cidr 0.0.0.0/0

##### Create the Fargate service
This command creates a Fargate service using the task definition which we registered before. Use the security GroupId from above. Also use your subnet id's from your default VPC in the command below.
 
    aws ecs create-service --region eu-west-1 --service-name javalin-service --task-definition sample-fargate --desired-count 1 --launch-type "FARGATE" --network-configuration "awsvpcConfiguration={subnets=[subnet-fcc5779b,subnet-f7a318be],securityGroups=[sg-043ab03ec99b60369],assignPublicIp=ENABLED}" --cluster fargate

##### Configure secrets
In Secrets in the Settings section of your Github repository, configure the two secrets below with the credentials for an IAM user (which you can obtain from the AWS console):

    AWS_ACCESS_KEY_ID
    AWS_SECRET_ACCESS_KEY

##### Adjust the Github Action
In the sample application edit the file `.github/workflows/aws.yml` and change the `aws-region: eu-west-1` property to your region.

##### Deploy the sample service
Commit everything and push it. Open your repository on Github and change to the Actions tab. You should see the deployment of the sample service in progress.

##### Redeploy the sample service
On every push to the repository the sample application get's built, a new docker image created and pushed to the ECR repository. The task definition get's updated with the new image. A new version of the task definition gets created and a rolling update of the service will be performed.

##### Test the sample service
Once successfully deployed we need to get the public IP adress. First list the tasks in the fargate cluster:

    aws ecs list-tasks  --cluster fargate

Next get information about the running task:

    aws ecs describe-tasks --task "arn:aws:ecs:eu-west-1:601912882130:task/fargate/e9551bfe073e435e803e22268f6b21d1" --cluster fargate

In the output of the above command you will find the "networkInterfaceId".

       {
            "name": "networkInterfaceId",
            "value": "eni-0a9e3072cf3299729"
        }

With the above networkInterfaceId you can find the public IP:

    aws ec2 describe-network-interfaces --network-interface-ids eni-0a9e3072cf3299729

In the output of the above command you will find the public IP adress:

    "PublicDnsName": "ec2-34-247-74-194.eu-west-1.compute.amazonaws.com",
    "PublicIp": "34.247.74.194"

Now we can test the service with curl. You should get "Hello World" as response.

    curl 34.247.74.194:7000
    Hello World

##### Cleaning Up
The AWS resources we created are not free. So to clean them up, we can run the following:

    aws ecs delete-service --service javalin-service --cluster fargate --force

##### Summary
In this post i demonstrated AWS Fargate. With a few commands we created a Fargate Cluster and a Service with one Task. And with the help of Github Actions we could deploy the service fully automated.

{{< figure src="https://bluehost-cdn.com/media/partner/images/antoniodol/488x160/488x160BW.png" target="_blank" link="https://www.bluehost.com/track/antoniodol/blue1" >}}


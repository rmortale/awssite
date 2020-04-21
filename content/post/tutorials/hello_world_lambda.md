---
title: "Hello world Lambda"
date: 2020-01-16T16:11:59+01:00
draft: false
author: Antonio
image: images/lambdablog.png
description: Create and deploy simple Lambda function with SAM
categories: 
  - Lambda
---

In this post we will use AWS Serverless Application Model (SAM) to create and deploy a simple Lambda function written in Java.

[![AWS](https://static.shareasale.com/image/43514/468X6010.jpg)](https://shareasale.com/r.cfm?b=1373702&amp;u=2310472&amp;m=43514&amp;urllink=&amp;afftrack=)

### Prerequisites
* AWS CLI
* Java 8 JDK installed
* Maven installed
* SAM CLI installed
* Docker installed


### Create project
    sam init -r java8 -n hello_world_lambda

### Build and test local
Change to the newly created directory `hello_world_lambda`.

    sam build
    sam local invoke --event events/event.json

### Package and deploy
    sam package --output-template-file packaged.yaml --s3-bucket existing_bucket_name
    sam deploy --template-file packaged.yaml --stack-name hello-world-lambda --capabilities CAPABILITY_IAM

### Test remote
Get the API Gateway endpoint url

    aws cloudformation describe-stacks --stack-name hello-world-lambda

Invoke API 
    
    curl endpoint url

### Delete
    aws cloudformation delete-stack --stack-name hello-world-lambda

{{< figure src="https://bluehost-cdn.com/media/partner/images/antoniodol/488x160/488x160BW.png" target="_blank" link="https://www.bluehost.com/track/antoniodol/blue1" >}}

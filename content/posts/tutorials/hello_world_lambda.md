---
title: "Hello world Lambda"
date: 2020-01-16T16:11:59+01:00
draft: false
---

In this post we will use AWS Serverless Application Model (SAM) to create and deploy a simple Lambda function written in Java.

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

### Books
Prepare for your exam with this book [AWS Certified SysOps Administrator - Associate (SOA-C01) Cert Guide](https://amzn.to/2tx3iDW) or [DVD](https://amzn.to/2TOdHpe)


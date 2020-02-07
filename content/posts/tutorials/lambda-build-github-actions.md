---
title: "Build and deploy Lambda with Github Action"
date: 2020-02-07T16:11:59+01:00
draft: false
---

In this post we will use AWS Serverless Application Model (SAM) to create and deploy a simple Lambda function written in Python. The Function get's built and deployed with [Github Actions](https://github.com/marketplace?type=actions).

### Prerequisites
* AWS CLI
* Python 3
* Git client
* SAM CLI installed
* Docker installed


### Create project
    sam init -r python3.8 -n pyth38-demo

### Build and test local
Change to the newly created directory `pyth38-demo`.

    sam build
    sam local invoke --event events/event.json

### Import the project into Github
Create a new repository in Github. Import the project in the new repository.

### Create Github Action
Go ot the repositories Settings and add your AWS access key (AWS_ACCESS_KEY_ID) and secret (AWS_SECRET_ACCESS_KEY) as Secrets.
Create a blank Github Action in your new repository with this code.

{{< highlight go "linenos=table,hl_lines=23" >}}
on: [push]

jobs:
  aws_sam:
    runs-on: ubuntu-latest
    steps:

      - name: Checkout
        uses: actions/checkout@master

      - name: sam build
        uses: youyo/aws-sam-action/python3.8@master
        with:
          sam_command: build
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION: eu-west-1

      - name: sam deploy
        uses: youyo/aws-sam-action/python3.8@master
        with:
          sam_command: 'deploy --stack-name pyth38-demo --s3-bucket bucket_name --capabilities CAPABILITY_IAM --no-fail-on-empty-changeset'
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION: eu-west-1
{{< / highlight >}}

Change the S3 bucket name to an existing bucket in the same region as your Lambda. Commit your changes. The Lambda Function will be built and deployed to your AWS account.


### Test remote
Get the API Gateway endpoint url

    aws cloudformation describe-stacks --stack-name pyth38-demo

Invoke API 
    
    curl endpoint url

### Delete
    aws cloudformation delete-stack --stack-name pyth38-demo

### Books
Prepare for your exam with this book [AWS Certified SysOps Administrator - Associate (SOA-C01) Cert Guide](https://amzn.to/2tx3iDW) or [DVD](https://amzn.to/2TOdHpe)


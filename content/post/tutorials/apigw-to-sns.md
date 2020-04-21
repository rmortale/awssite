---
title: API Gateway and asynchronous microservices
date: 2020-04-12T20:20:16+01:00
draft: false
author: Antonio
image: images/API-Gateway-to-SNS.png
description: How to connect API Gateway to SNS
categories: 
  - API Gateway
  - SNS
  - CloudFormation
---

In this tutorial we will create an API Gateway which is directly connected to a SNS Topic. To this Topic we subscribe one or more SQS queues (see the picture above). To this queues we can connect our microservices (not covered in this tutorial). This simple architecture enables the user to submit an asynchronous request thru the API Gateway and to process this request in a scalable way.

[![AWS](https://static.shareasale.com/image/43514/468X6010.jpg)](https://shareasale.com/r.cfm?b=1373702&amp;u=2310472&amp;m=43514&amp;urllink=&amp;afftrack=)

#### Prerequisites
* Git client
* AWS Account
* AWS CLI configured

#### Clone the tutorial repository
Clone the tutorial [repository](https://github.com/rmortale/cfn-apigw-tut) from Github. Change to the newly created directory.

#### Create the resources
Create the resources with this command:

    aws cloudformation create-stack --stack-name orderapi --template-body file://apigwsns.yaml --capabilities CAPABILITY_IAM

#### Test the stack
Wait until the Stack is deployed. Then get the API Gateway URL.

    aws cloudformation describe-stacks --stack-name orderapi
    "OutputKey": "APIDomainNameWithStage",
    "OutputValue": "https://r96ckdi3ph.execute-api.eu-west-1.amazonaws.com/prod"

Test the API with curl.

    curl -X POST https://r96ckdi3ph.execute-api.eu-west-1.amazonaws.com/prod/orders   --data 'Order data' -H 'Content-Type: application/json'

    {"body": "Order received."}

Now check if we got the order message in the SQS queue. Get the QueueURL first:

    aws cloudformation describe-stacks --stack-name orderapi
    "OutputKey": "QueueURL",
    "OutputValue": "https://sqs.eu-west-1.amazonaws.com/601912882130/orderapi-OrderServiceQueue-V5O42JJ01TTF"

Check the message in the queue:

    aws sqs receive-message --queue-url https://sqs.eu-west-1.amazonaws.com/601912882130/orderapi-OrderServiceQueue-V5O42JJ01TTF

    "Messages": [
        {
            "MessageId": "d3904be5-fb9b-446f-ab86-d7540c2fb2a1",.....

Success! We got the order message in the queue. Now on this queue would listening a microservice (Lambda, ECS or other) and process the message. But this is something for another tutorial.

#### Summary
In this tutorial we created an API Gateway which is directly connected to a SNS Topic. To this Topic we subscribed one SQS queue.

{{< figure src="https://bluehost-cdn.com/media/partner/images/antoniodol/488x160/488x160BW.png" target="_blank" link="https://www.bluehost.com/track/antoniodol/blue1" >}}

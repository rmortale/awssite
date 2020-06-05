---
title: "Deploy Lambda function with Terraform"
date: 2020-05-03T11:11:59+01:00
draft: false
author: Antonio
image: images/lambdablog.png
description: Deploy an AWS Lambda function using Terraform only.
categories: 
  - Lambda
  - Terraform
---

In this post we will use [Terraform](https://www.terraform.io/) to deploy a simple Lambda function written in Python. You can follow the video or continue reading.

[![AWS](https://static.shareasale.com/image/43514/300X2503_00.jpg)](https://shareasale.com/r.cfm?b=1551034&amp;u=2310472&amp;m=43514&amp;urllink=&amp;afftrack=)

{{< youtube 6FA3h7_0-68 >}}

### Prerequisites
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv1.html) installed and configured
* [Terraform CLI](https://learn.hashicorp.com/terraform/getting-started/install.html#install-terraform) installed


### Clone the tutorial repository
Clone the tutorial [repository](https://github.com/rmortale/terraform-tutorial) from Github. Change to the newly created directory.

### Initialize Terraform
    terraform init

This will install and initialize the AWS Provider.

### Deploy
    terraform apply

This will package the Python handler and create the Lambda function.

### Test
    aws lambda invoke --function-name hello_lambda out.txt

This command invokes the Lambda function and should return with:

    {
        "StatusCode": 200,
        "ExecutedVersion": "$LATEST"
    }

The output is in the out.txt file and should be:
    
    "Hello from Lambda!"


### Delete
To delete all resources created use:

    terraform destroy

{{< figure src="https://bluehost-cdn.com/media/partner/images/antoniodol/488x160/488x160BW.png" target="_blank" link="https://www.bluehost.com/track/antoniodol/blue1" >}}

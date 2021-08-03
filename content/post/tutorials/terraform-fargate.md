---
title: "Create a Fargate Service with Terraform"
date: 2020-06-05T09:11:59+01:00
draft: false
author: Antonio
image: images/fargate2.png
description: Create a Fargate Service using Terraform.
categories: 
  - ECS
  - Terraform
  - Fargate
---

In this post we will use [Terraform](https://www.terraform.io/) to create a Fargate Cluster and deploy a service to it. You can follow the video or continue reading.

{{< youtube 97p8lI8MypI >}}

### Prerequisites
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv1.html) installed and configured
* [Terraform CLI](https://learn.hashicorp.com/terraform/getting-started/install.html#install-terraform) installed


### Create Terraform project
Create a new directory and create a file named `main.tf` in it. Copy/Paste this code in the file. You may adjust the region and availability_zones.

    provider "aws" {
        region = "eu-central-1"
    }

    module "base-network" {
        source                                      = "cn-terraform/networking/aws"
        version                                     = "2.0.7"
        name_preffix                                = "test-networking"
        vpc_cidr_block                              = "192.168.0.0/16"
        availability_zones                          = ["eu-central-1a", "eu-central-1b"]
        public_subnets_cidrs_per_availability_zone  = ["192.168.0.0/19", "192.168.32.0/19"]
        private_subnets_cidrs_per_availability_zone = ["192.168.128.0/19", "192.168.160.0/19"]
    }

    module "ecs-fargate" {
        source  = "cn-terraform/ecs-fargate/aws"
        version = "2.0.17"
        name_preffix        = "test"
        vpc_id              = module.base-network.vpc_id
        container_image     = "nginx"
        container_name      = "test"
        public_subnets_ids  = module.base-network.public_subnets_ids
        private_subnets_ids = module.base-network.private_subnets_ids
        lb_enable_https = false
    }

Create a second file named `outputs.tf` and copy/paste this lines to it.

    output "loadbalancer-address" {
        value = "${module.ecs-fargate.aws_lb_lb_dns_name}"
    }

### Initialize Terraform
    terraform init

This will install and initialize the AWS Provider.

### Deploy
The AWS resources we create are not free. Be sure to clean them up at the end!

    terraform apply

This will create a VPC with public and private subnets. A Fargate Cluster. A load balancer. A Fargate Task and Service with one nginx container. At the end you will see

    ...
    Apply complete! Resources: 41 added, 0 changed, 0 destroyed.

    Outputs:

    loadbalancer-address = test-lb-1227991415.eu-central-1.elb.amazonaws.com

### Test
You may have to wait some seconds until the nginx service is ready. Then copy the Loadbalancer address from the outputs section above and test the service with:

    curl test-lb-1227991415.eu-central-1.elb.amazonaws.com

You should see the default nginx page

    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>


### Delete
To delete all resources created use:

    terraform destroy

### Summary
In this post we created a Fargate cluster and deployed a service to it.


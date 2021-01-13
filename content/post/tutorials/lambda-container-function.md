---
title: "AWS Lambda function using a container image"
date: 2021-01-09T17:11:59+01:00
draft: false
author: Antonio
image: images/lambdablog.png
description: Package a Lambda function as container image.
categories: 
  - Lambda
  - Docker
---

You can package your Lambda function code and dependencies as a container image, using tools such as the Docker CLI. You can then upload the image to your container registry hosted on Amazon Elastic Container Registry (Amazon ECR) and deploy it to AWS Lambda. You can follow the video or continue reading.

{{< youtube UiVOj6rR6Yk >}}

[![Certified Kubernetes Administrator](https://static.shareasale.com/image/43514/Certified_Kubernetes_Administrator.jpg)](https://shareasale.com/r.cfm?b=1543562&amp;u=2310472&amp;m=43514&amp;urllink=&amp;afftrack=)

### Prerequisites
* [Docker](https://docs.docker.com/get-docker/) installed
* Java JDK 11
* Amazon AWS Account
* AWS CLI
* [SAM CLI](https://docs.aws.amazon.com/serverless-application-model/index.html)

### Create the sample project
We use the SAM CLI to create a sample Lambda project.

    PS C:\Users\Nino\code\aws\sam-app> sam --version
    SAM CLI, version 1.15.0

Type `sam init` to get started!

    PS C:\Users\Nino\code\aws> sam init
    Which template source would you like to use?
            1 - AWS Quick Start Templates
            2 - Custom Template Location
    Choice: 1
    What package type would you like to use?
            1 - Zip (artifact is a zip uploaded to S3)
            2 - Image (artifact is an image uploaded to an ECR image repository)
    Package type: 2

    Which base image would you like to use?
            1 - amazon/nodejs12.x-base
            2 - amazon/nodejs10.x-base
            3 - amazon/python3.8-base
            4 - amazon/python3.7-base
            5 - amazon/python3.6-base
            6 - amazon/python2.7-base
            7 - amazon/ruby2.7-base
            8 - amazon/ruby2.5-base
            9 - amazon/go1.x-base
            10 - amazon/java11-base
            11 - amazon/java8.al2-base
            12 - amazon/java8-base
            13 - amazon/dotnetcore3.1-base
            14 - amazon/dotnetcore2.1-base
    Base image: 10

    Which dependency manager would you like to use?
            1 - maven
            2 - gradle
    Dependency manager: 1

    Project name [sam-app]:

    Cloning app templates from https://github.com/aws/aws-sam-cli-app-templates

        -----------------------
        Generating application:
        -----------------------
        Name: sam-app
        Base Image: amazon/java11-base
        Dependency Manager: maven
        Output Directory: .

        Next steps can be found in the README file at ./sam-app/README.md

### Build the sample project
Change to the sample project directory and build the application.

    PS C:\Users\Nino\code\aws> cd .\sam-app\
    sam build

After some seconds your Docker image should be ready.

    ...
    Successfully built 02c1ac70770e
    Successfully tagged helloworldfunction:java11-maven-v1

    Build Succeeded

    Built Artifacts  : .aws-sam\build
    Built Template   : .aws-sam\build\template.yaml

### Test locally
Use `sam local invoke` to test the function locally.

    PS C:\Users\Nino\code\aws\sam-app> sam local invoke
    Invoking Container created from helloworldfunction:java11-maven-v1
    Building image.........
    Skip pulling image and use local one: helloworldfunction:rapid-1.15.0.

    START RequestId: 9e5e7829-cac0-49b5-a779-cbfceed96b90 Version: $LATEST
    END RequestId: 9e5e7829-cac0-49b5-a779-cbfceed96b90
    REPORT RequestId: 9e5e7829-cac0-49b5-a779-cbfceed96b90  Init Duration: 0.10 ms  Duration: 1323.79 ms    Billed Duration: 1400 ms        Memory Size: 128 MB     Max Memory Used: 128 MB
    {"statusCode":200,"headers":{"X-Custom-Header":"application/json","Content-Type":"application/json"},"body":"{ \"message\": \"hello world\", \"location\": \"xx.xx.xx.xx\" }"}

### Create a Container Registry
Before we can continue we need to create a Amazon Elastic Container Registry (Amazon ECR) and login to it.

    aws ecr create-repository --repository-name lambda-sample
    {
        "repository": {
            "repositoryArn": "arn:aws:ecr:eu-central-1:601912882130:repository/lambda-sample",
            "registryId": "601912882130",
            "repositoryName": "lambda-sample",
            "repositoryUri": "601912882130.dkr.ecr.eu-central-1.amazonaws.com/lambda-sample",
            "createdAt": "2021-01-09T18:20:40+01:00",
            "imageTagMutability": "MUTABLE",
            "imageScanningConfiguration": {
                "scanOnPush": false
            },
            "encryptionConfiguration": {
                "encryptionType": "AES256"
            }
        }
    }

In the output above you find your `repositoryUri` which we need later when we deploy the function. Next we need to authenticate the Docker CLI to the registry.
Use your region and `repositoryUri` without the repository name.

    aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin 601912882130.dkr.ecr.eu-central-1.amazonaws.com
    Login Succeeded

### Deploy the project
To deploy the project we use `sam deploy --guided`. Again use your region and `repositoryUri`.

    PS C:\Users\Nino\code\aws\sam-app> sam deploy --guided

    Configuring SAM deploy
    ======================

            Looking for config file [samconfig.toml] :  Not found

            Setting default arguments for 'sam deploy'
            =========================================
            Stack Name [sam-app]:
            AWS Region [us-east-1]: eu-central-1
            Image Repository for HelloWorldFunction: 601912882130.dkr.ecr.eu-central-1.amazonaws.com/lambda-sample
            helloworldfunction:java11-maven-v1 to be pushed to 601912882130.dkr.ecr.eu-central-1.amazonaws.com/lambda-sample:helloworldfunction-02c1ac70770e-java11-maven-v1

            #Shows you resources changes to be deployed and require a 'Y' to initiate deploy
            Confirm changes before deploy [y/N]: y
            #SAM needs permission to be able to create roles to connect to the resources in your template
            Allow SAM CLI IAM role creation [Y/n]:
            HelloWorldFunction may not have authorization defined, Is this okay? [y/N]: y
            Save arguments to configuration file [Y/n]:
            SAM configuration file [samconfig.toml]:
            SAM configuration environment [default]:

            Looking for resources needed for deployment: Found!

                    Managed S3 bucket: aws-sam-cli-managed-default-samclisourcebucket-o9p6qnwsutlu
                    A different default S3 bucket can be set in samconfig.toml

            Saved arguments to config file
            Running 'sam deploy' for future deployments will use the parameters saved above.
            The above parameters can be changed by modifying samconfig.toml
            Learn more about samconfig.toml syntax at
            https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-config.html

    The push refers to repository [601912882130.dkr.ecr.eu-central-1.amazonaws.com/lambda-sample]
    d6a934891197: Pushed
    db00620364e4: Pushed
    8d84955f4cab: Pushed
    4ec3f54f3699: Pushed
    d6fa53d6caa6: Pushed
    88f673ad6865: Pushed
    ce46efaa45fc: Pushed
    f2342b1247df: Pushed
    helloworldfunction-02c1ac70770e-java11-maven-v1: digest: sha256:860264755d6fd8c44791a727186bca389d83e6290f53b009d605dbaac5bb9959 size: 1998


            Deploying with following values
            ===============================
            Stack name                   : sam-app
            Region                       : eu-central-1
            Confirm changeset            : True
            Deployment image repository  :
                                        {
                                            "HelloWorldFunction": "601912882130.dkr.ecr.eu-central-1.amazonaws.com/lambda-sample"
                                        }
            Deployment s3 bucket         : aws-sam-cli-managed-default-samclisourcebucket-o9p6qnwsutlu
            Capabilities                 : ["CAPABILITY_IAM"]
            Parameter overrides          : {}
            Signing Profiles             : {}

    Initiating deployment
    =====================
    HelloWorldFunction may not have authorization defined.
    Uploading to sam-app/1620aece466b1548be891f4dcff345ca.template  1222 / 1222.0  (100.00%)

    Waiting for changeset to be created..

    CloudFormation stack changeset
    -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    Operation                                      LogicalResourceId                              ResourceType                                   Replacement
    -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    + Add                                          HelloWorldFunctionHelloWorldPermissionProd     AWS::Lambda::Permission                        N/A
    + Add                                          HelloWorldFunctionRole                         AWS::IAM::Role                                 N/A
    + Add                                          HelloWorldFunction                             AWS::Lambda::Function                          N/A
    + Add                                          ServerlessRestApiDeployment47fc2d5f9d          AWS::ApiGateway::Deployment                    N/A
    + Add                                          ServerlessRestApiProdStage                     AWS::ApiGateway::Stage                         N/A
    + Add                                          ServerlessRestApi                              AWS::ApiGateway::RestApi                       N/A
    -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

    Changeset created successfully. arn:aws:cloudformation:eu-central-1:601912882130:changeSet/samcli-deploy1610215043/9ca23992-2a71-4d24-80b3-1b814977ce70


    Previewing CloudFormation changeset before deployment
    ======================================================
    Deploy this changeset? [y/N]: y
    ...
    CloudFormation outputs from deployed stack
    ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    Outputs
    ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    Key                 HelloWorldFunctionIamRole
    Description         Implicit IAM Role created for Hello World function
    Value               arn:aws:iam::601912882130:role/sam-app-HelloWorldFunctionRole-10CJ3JI7E28HN

    Key                 HelloWorldApi
    Description         API Gateway endpoint URL for Prod stage for Hello World function
    Value               https://3l55ggqaqd.execute-api.eu-central-1.amazonaws.com/Prod/hello/

    Key                 HelloWorldFunction
    Description         Hello World Lambda Function ARN
    Value               arn:aws:lambda:eu-central-1:601912882130:function:sam-app-HelloWorldFunction-IA5LHYN3NRJH
    ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

    Successfully created/updated stack - sam-app in eu-central-1


### Test the function
From the output above get the `API Gateway endpoint URL` to test the function with curl.

    PS C:\Users\Nino\code\aws\sam-app> curl https://3l55ggqaqd.execute-api.eu-central-1.amazonaws.com/Prod/hello/


    StatusCode        : 200
    StatusDescription : OK
    Content           : { "message": "hello world", "location": "xx.xxx.xx.xxx" }
    RawContent        : HTTP/1.1 200 OK
                        Connection: keep-alive
                        x-amzn-RequestId: 88a4f701-7ab9-44a9-8084-cdd223fe0dcd
                        x-amz-apigw-id: Y5J00Eu6liAFnrg=
                        X-Custom-Header: application/json
                        X-Amzn-Trace-Id: Root=1-5ff9f01e-1...
    Forms             : {}
    Headers           : {[Connection, keep-alive], [x-amzn-RequestId, 88a4f701-7ab9-44a9-8084-cdd223fe0dcd], [x-amz-apigw-id, Y5J00Eu6liAFnrg=], [X-Custom-Header, application/json]...}
    Images            : {}
    InputFields       : {}
    Links             : {}
    ParsedHtml        : System.__ComObject
    RawContentLength  : 56

Successfull! We got a 200 status code.

### Delete the lambda project
To delete the created resources use:

    aws cloudformation delete-stack --stack-name sam-app
    aws ecr delete-repository --repository-name lambda-sample --force

### Summary
In this post we created a sample Lambda project, packaged our Lambda function code and dependencies as a container image, deployed the container image and tested it successfully.

{{< figure src="https://bluehost-cdn.com/media/partner/images/antoniodol/488x160/488x160BW.png" target="_blank" link="https://www.bluehost.com/track/antoniodol/blue1" >}}

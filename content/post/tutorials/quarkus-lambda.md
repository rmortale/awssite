---
title: "AWS Lambda custom runtime with Quarkus"
date: 2020-09-19T16:11:59+01:00
draft: false
author: Antonio
image: images/quarkus.jpg
description: The quarkus-amazon-lambda extension allows you to use Quarkus to build your AWS Lambdas.
categories: 
  - Lambda
  - Quarkus
---

The [quarkus-amazon-lambda extension](https://quarkus.io/guides/amazon-lambda) allows you to use [Quarkus](https://quarkus.io) to build your AWS Lambdas. Your Lambdas can use injection annotations from CDI or Spring and other Quarkus facilities as you need them.

Quarkus Lambdas can be deployed using the Amazon Java Runtime, or you can build a native executable and use Amazon’s Custom Runtime if you want a smaller memory footprint and faster cold boot startup time.

[![Certified Kubernetes Administrator](https://static.shareasale.com/image/43514/Certified_Kubernetes_Administrator.jpg)](https://shareasale.com/r.cfm?b=1543562&amp;u=2310472&amp;m=43514&amp;urllink=&amp;afftrack=)

### Prerequisites
* [Docker](https://docs.docker.com/get-docker/) installed
* Java JDK 11
* [Apache Maven 3.6.2+](http://maven.apache.org/)
* Amazon AWS Account
* AWS CLI and SAM CLI

### Create the sample project
We use the `quarkus-amazon-lambda-archetype` to create our sample maven project. Give the project a artifactId of `quarkus-lambda`.

    mvn archetype:generate -DarchetypeGroupId=io.quarkus -DarchetypeArtifactId=quarkus-amazon-lambda-archetype -DarchetypeVersion=1.8.0.Final

    cd quarkus-lambda


### Lambda Handler
The Lambda Handler class is in `src/main/java/yourpackagename/TestLambda.java`. As you can see the class implements the `RequestHandler` interface from the AWS SDK.

    package ch.dulce;

    import javax.inject.Inject;
    import javax.inject.Named;

    import com.amazonaws.services.lambda.runtime.Context;
    import com.amazonaws.services.lambda.runtime.RequestHandler;

    @Named("test")
    public class TestLambda implements RequestHandler<InputObject, OutputObject> {

        @Inject
        ProcessingService service;

        @Override
        public OutputObject handleRequest(InputObject input, Context context) {
            return service.process(input).setRequestId(context.getAwsRequestId());
        }
    }

### Build and create the Lambda function
First we need to create the iam role and policy for our function. The first command returns the role arn which we need later when we create the function.

    aws iam create-role --role-name lambda-ex --assume-role-policy-document "{\"Version\": \"2012-10-17\",\"Statement\": [{ \"Effect\": \"Allow\", \"Principal\": {\"Service\": \"lambda.amazonaws.com\"}, \"Action\": \"sts:AssumeRole\"}]}"
    
    aws iam attach-role-policy --role-name lambda-ex --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

    set LAMBDA_ROLE_ARN="arn from the first command above"

Now let us build the sample project. We use a docker image to create the native Linux executable.

    mvn clean install -Pnative -Dnative-image.docker-build=true

After the build we have a `function.zip` file in the `target` directory with our Lambda function. With the next command we execute a local test of the function.

    sam local invoke --template target\sam.native.yaml --event payload.json 

The result should look like this

    {"result":"hello Bill","requestId":"f19005f2-603d-126b-3f2a-037b5b1fab6a"}

After testing we create the function

    aws lambda create-function ^
        --function-name QuarkusLambdaNative ^
        --zip-file fileb://./target\function.zip ^
        --handler na ^
        --runtime provided ^
        --role %LAMBDA_ROLE_ARN% ^
        --timeout 15 ^
        --memory-size 256 ^
        --environment Variables={DISABLE_SIGNAL_HANDLERS=true}

After the above command returned sucessfully, we can invoke the function

    aws lambda invoke --function-name QuarkusLambdaNative --payload file://payload.json --cli-binary-format raw-in-base64-out response.txt

The output from the function call looks like this

    type response.txt
    {"result":"hello Bill","requestId":"8e788814-9602-485f-a088-d83cb5983709"}

### Delete
Delete the function with the command

    aws lambda delete-function --function-name QuarkusLambdaNative

### Summary
In this post we used Quarkus to build a Lambda function with `custom` runtime. The advantage of this is a fast startup and execution time of the Lambda function.

{{< figure src="https://bluehost-cdn.com/media/partner/images/antoniodol/488x160/488x160BW.png" target="_blank" link="https://www.bluehost.com/track/antoniodol/blue1" >}}

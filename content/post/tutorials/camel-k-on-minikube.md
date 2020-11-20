---
title: Install Camel K on minikube
date: 2020-11-18T20:20:16+01:00
draft: false
author: Antonio
image: images/camel.png
description: How to install Apache Camel K on minikube
categories: 
  - minikube
  - Camel K
---

In this tutorial we install [Apache Camel K](https://camel.apache.org/camel-k/latest/) on [minikube](https://minikube.sigs.k8s.io/docs/). Apache Camel K is a lightweight integration framework built from Apache Camel that runs natively on Kubernetes and is specifically designed for serverless and microservice architecture. You can follow the video or continue reading.

{{< youtube UH_jkoW4VjI >}}

#### Prerequisites
* [Docker installed](https://docs.docker.com/get-docker/)
* [Minikube installed]({{< ref "install-minikube" >}})

#### Download and install camel-k cli
Download the camel-k cli binary for your Operating System from [here](https://github.com/apache/camel-k/releases). I am using version 1.2.0. You put the binary on your path.

#### Start the cluster
I am using the Docker driver for this Tutorial. I start minikube with this command:

    minikube start --driver=docker

After a minute or so the cluster should be up and running. If minikube fails to start, see the [drivers](https://minikube.sigs.k8s.io/docs/drivers/) page for help setting up a compatible container or virtual-machine manager.

Add the `registry` addon to the minikube cluster.

    minikube addons enable registry

#### Install camel-k
To install the camel-k operator type

    kamel install

Test the installation with the command

    kamel get
    NAME    PHASE   KIT

You get an empty list of integrations. Let's change this.

#### Deploy the sample code
Download the sample code from [here](https://github.com/rmortale/camel-exc-demo/tags) and unzip it. Change to the unzipped directory. 

#### Start activemq docker container
For testing we need a running Apache ActiveMQ instance. We use an existing docker image and start the container like this

        docker run -p 61616:61616 -p 8161:8161 rmohr/activemq

Access the [admin console](http://localhost:8161) and login with de default user `admin` and password `admin`.

The sample code looks like this

        public class ActivemqTransacted extends RouteBuilder {

            @Override
            public void configure() {

                from("activemq:inqueue").throwException(IllegalArgumentException.class, "bad message");
                
            }

        }

We listen on a queue named `inqueue` for messages and throw immedietly an `IllegalArgumentException`. If we send a message to the `inqueue` the `DefaultErrorHandler` kicks in and the exception will be propagated back to the caller and the Exchange ends immediately. Let's have a look on the `application.properties` file. Please replace the IP Adresse in the `broker-url` with your IP.

        # transactionsautowired props
        camel.component.activemq.broker-url=tcp://192.168.0.129:61616
        camel.component.activemq.transacted=false
        
Now we deploy the camel-k integration with this command

        kamel run --dev --property-file src/main/resources/application.properties src/main/java/ch/dulce/ActivemqTransacted.java

The integration get's built and deployed to our minikube instance. Access the [admin console](http://localhost:8161) and login with de default user `admin` and password `admin`. Go to the `Queues` tab. We should see a queue named `inqueue`. Klick on the right side under `Operations` on `Send To` and send a test message. Enable `Persistent Delivery` like this

![send message to queue](/images/activemqSendTo.PNG)

Click on the send button. We see the `IllegalArgumentException` thrown and the message get's consumed (lost)! Now let's enable the transacted attribute in the `application.properties` file.

        # transactionsautowired props
        camel.component.activemq.broker-url=tcp://192.168.0.129:61616
        camel.component.activemq.transacted=true <- set to true

The integration should get redeployed automatically since we used the `kamel --dev` flag. Send another test message to the `inqueue`. This time the message is not lost because activemq rolls back the transaction on the `IllegalArgumentException` and redelivers the message until the activemq broker default redelivery times (6) is reached. Then the message will be sent to a Dead Letter Queue (DLQ).

![message in DLQ](/images/activemqDlq.PNG)

Type `Ctrl-C` in the window where you executed the `kamel run` command to undeploy the integration.

#### Summary
In this tutorial we installed camel-k on minikube. And made some test wit a simple integration using activemq with transactions.


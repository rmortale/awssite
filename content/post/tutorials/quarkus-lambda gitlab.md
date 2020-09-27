---
title: "AWS Lambda with Quarkus on Gitlab CI/CD"
date: 2020-09-27T16:11:59+01:00
draft: false
author: Antonio
image: images/quarkus.jpg
description: The quarkus-amazon-lambda maven extension allows you to use Quarkus to build your AWS Lambdas.
categories: 
  - Lambda
  - Quarkus
  - Gitlab
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
* [Gitlab Account](https://gitlab.com)

### Create the sample project
We use the `quarkus-amazon-lambda-archetype` to create our sample maven project. Give the project a artifactId of `quarkus-lambda-gitlab`.

    mvn archetype:generate -DarchetypeGroupId=io.quarkus -DarchetypeArtifactId=quarkus-amazon-lambda-archetype -DarchetypeVersion=1.8.0.Final

    cd quarkus-lambda-gitlab

    # remove gradle files
    rm build.gradle settings.gradle gradle.properties


### Create Gitlab project
Go to `gitlab.com` login with your account. Create new project. Name it `quarkus-lambda-gitlab`. Change to your local maven project and enter:

    git init
    git remote add origin https://gitlab.com/youraccount/quarkus-lambda-gitlab.git
    git add .
    git commit -m "Initial commit"
    git push -u origin master

### Prepare project for Gitlab maven repository
Change jdk version in `pom.xml` from 1.8 to 11

    <maven.compiler.target>11</maven.compiler.target>
    <maven.compiler.source>11</maven.compiler.source>

Add a new file in the root of your project named `system.properties` with this line:

    java.runtime.version=11

Add this line to the `src/main/resources/application.properties` file

    quarkus.lambda.enable-polling-jvm-mode=true

### Create a personal access token
Follow this [instructions](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html) to create a personal access token with the `api` scope.

Add the token to your maven `settings.xml` file. The file should be in your `$HOME/.m2` directory. If not, create the file.

    <settings>
        <servers>
            <server>
                <id>gitlab-maven</id>
                <configuration>
                    <httpHeaders>
                        <property>
                            <name>Private-Token</name>
                            <value>REPLACE_WITH_YOUR_PERSONAL_ACCESS_TOKEN</value>
                        </property>
                    </httpHeaders>
                </configuration>
            </server>
        </servers>
    </settings>

Add the repository settings to your `pom.xml`. Change the `PROJECT_ID` with your project id which you find on your project overview page in Gitlab.

    <repositories>
        <repository>
            <id>gitlab-maven</id>
            <url>https://gitlab.com/api/v4/projects/PROJECT_ID/packages/maven</url>
        </repository>
    </repositories>
    <distributionManagement>
        <repository>
            <id>gitlab-maven</id>
            <url>https://gitlab.com/api/v4/projects/PROJECT_ID/packages/maven</url>
        </repository>
        <snapshotRepository>
            <id>gitlab-maven</id>
            <url>https://gitlab.com/api/v4/projects/PROJECT_ID/packages/maven</url>
        </snapshotRepository>
    </distributionManagement>

Add the assembly plugin to the `pom.xml`

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>3.1.0</version>
        <configuration>
            <descriptors>src/main/assembly/assembly.xml</descriptors>
        </configuration>
        <executions>
            <execution>
                <phase>verify</phase>
                <goals>
                    <goal>single</goal>
                </goals>
            </execution>
        </executions>
    </plugin>

And the assembly descriptor to `src/main/assembly/assembly.xml`. This will add the native Quarkus binary to a zip file.

    <assembly xmlns="http://maven.apache.org/ASSEMBLY/2.1.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/ASSEMBLY/2.1.0 http://maven.apache.org/xsd/assembly-2.1.0.xsd">
        <id>bin</id>
        <formats>
            <format>zip</format>
        </formats>
        <includeBaseDirectory>false</includeBaseDirectory>
        <files>
            <file>
                <source>${project.build.directory}/${project.build.finalName}-runner</source>
                <destName>bootstrap</destName>
            </file>
        </files>
    </assembly>

### Deploy the project
To deploy the project to the Gitlab maven repository enter:

     mvn clean deploy -Pnative -Dquarkus.native.container-build=true

After the last command finished successfully the Lambda function is zipped and deployed to the maven repository.

### Prepare CI pipeline

Create a `ci_settings.xml` file in your project root directory with this content

    <settings xmlns="http://maven.apache.org/SETTINGS/1.1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.1.0 http://maven.apache.org/xsd/settings-1.1.0.xsd">
        <servers>
            <server>
                <id>gitlab-maven</id>
                <configuration>
                    <httpHeaders>
                        <property>
                            <name>Job-Token</name>
                            <value>${env.CI_JOB_TOKEN}</value>
                        </property>
                    </httpHeaders>
                </configuration>
            </server>
        </servers>
    </settings>

Add a shell script named `install-graal.sh` to your project root directory with this content

    #!/usr/bin/env bash
 
    echo "Downloading GraalVM"
    curl -L https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-20.1.0/graalvm-ce-java11-linux-amd64-20.1.0.tar.gz > graalvm-ce-java11-linux-amd64-20.1.0.tar.gz
    tar -zxf graalvm-ce-java11-linux-amd64-20.1.0.tar.gz -C ${CI_PROJECT_DIR}/
    ls -la
    
    echo "Installing GraalVM via gu"
    
    ${CI_PROJECT_DIR}/graalvm-ce-java11-20.1.0/bin/gu install native-image

Add a pipeline script named `.gitlab-ci.yml` to your project root directory with this content

    image: maven:3.6.3-openjdk-11-slim

    variables:
    # This will supress any download for dependencies and plugins or upload messages which would clutter the console log.
    # `showDateTime` will show the passed time in milliseconds. You need to specify `--batch-mode` to make this work.
    MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository -Dorg.slf4j.simpleLogger.log.org.apache.maven.cli.transfer.Slf4jMavenTransferListener=WARN -Dorg.slf4j.simpleLogger.showDateTime=true -Djava.awt.headless=true"
    # As of Maven 3.3.0 instead of this you may define these options in `.mvn/maven.config` so the same config is used
    # when running from the command line.
    # `installAtEnd` and `deployAtEnd` are only effective with recent version of the corresponding plugins.
    MAVEN_CLI_OPTS: "--batch-mode --errors --fail-at-end --show-version -DinstallAtEnd=true -DdeployAtEnd=true"

    cache:
    paths:
        - .m2/repository

    stages:
    - build
    - release
    - deploy

    before_script:
    - echo " ------------------------------- Global > Before Script -------------------------------"
    - echo $CI_COMMIT_BRANCH
    - apt-get update -qq
    - apt-get install -y -qq build-essential libz-dev zlib1g-dev

    build-native:
    stage: build
    script:
        - chmod +x install-graal.sh && ./install-graal.sh jdk11
        - export GRAALVM_HOME=${CI_PROJECT_DIR}/graalvm-ce-java11-20.1.0
        - mvn clean deploy -s ci_settings.xml -P native
    only:
        - master

Commit and push all changes. After some minutes you should have the Lambda function code built and zipped in your Gitlab maven repository.


### Summary
In this post we used Quarkus to build a Lambda function with `custom` runtime. The advantage of this is a fast startup and execution time of the Lambda function. We built a CI pipeline using Gitlab. In the next part we will deploy this Lambda function with help of terraform.

{{< figure src="https://bluehost-cdn.com/media/partner/images/antoniodol/488x160/488x160BW.png" target="_blank" link="https://www.bluehost.com/track/antoniodol/blue1" >}}

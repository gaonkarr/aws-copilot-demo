
# AWS Copilot Sample microservices demonstration


- [**Pre-requisites**](#pre-requisites)
- [**Install AWS Copilot**](#install-aws-copilot)
- [**Deploy Sample Application with ONE command only**](#deploy-sample-application-with-one-command-only)
- [**BUILD-Deploy Microservices Application**](#build-deploy-microservices-application)
- [**RELEASE-Deploy Release Pipeline for the Microservices Application**](#release-deploy-release-pipeline-for-the-microservices-application)
- [**OPERATE-Monitor Logs and Status**](#operate-monitor-logs-and-status)
- [**Clean up**](#clean-up)
- [**References and More**](#reference-and-more)


AWS Copilot is an open source command line interface that makes it easy for developers to **BUILD**,**RELEASE**, and **OPERATE** production ready containerized applications on AWS App Runner, Amazon ECS, and AWS Fargate.
You can start simple with just a Dockerfile and then grow with addons. You can work across multiple AWS accounts, and regions. Add more automation and have more visibility in your application’s health. 

Using AWS CloudFormation, it will automate all this so you can only work on building efficient containers and copilot will take care of How you build, release and operate on your code. It's an interactive way to deploy some standard application environments for you to quickly setup your microservices application. 

For example, a 'Load Balanced Web Service' would look something like this : 
![Load Balanced Web Service Architecture with AWS Copilot ](/images/Load-balanced-web-service-copilot.png)

## Pre-requisites
You will need to have the latest version of the AWS CLI installed and configured before running the deployment script. 

After installing the AWS CLI, simply run the 'aws configure' command once. This setups the default profile in your environment.

If you need help installing and configuring, please follow the links below:

[Installing the AWS CLI ](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)

[Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

The easiest way is to [create an AWS Cloud9 environment](https://docs.aws.amazon.com/cloud9/latest/user-guide/tutorial-create-environment.html#tutorial-create-environment-console) that comes pre-built with pre-requisites. Just run 'aws configure' once and you are good to go.


## Install AWS Copilot

Installing AWS Copilot is easy. Checkout the [manual instructions](https://aws.github.io/copilot-cli/docs/getting-started/install/) based on your Operating System in the Copilot Docs.

For Linux x86 (64-bit), use following command :

```shell
curl -Lo copilot https://github.com/aws/copilot-cli/releases/latest/download/copilot-linux && \
chmod +x copilot &&                                 \
sudo mv copilot /usr/local/bin/copilot &&           \
copilot --help &&                                 \
copilot --version
```

At the time of writing this Readme, copilot was at version: v1.8.0

## Deploy Sample Application with ONE command only

To deploy a sample Load Balanced Web Service application with just one command, run the following command :

```shell
git clone https://github.com/aws-samples/amazon-ecs-cli-sample-app.git demo-app && \ 
cd demo-app &&                               \
copilot init --app demo                      \
  --name api                                 \
  --type 'Load Balanced Web Service'         \
  --dockerfile './Dockerfile'                \
  --port 80                                  \
  --deploy
```

This will clone the AWS sample app, and initiate the deployment of the application. It will take few minutes for it to automatically create basic networking infrastructure, build your docker image, create a repository in Amazon ECR, push the docker image, create the Amazon ECS Clusters, ALB, and finally create tasks. In the end, it will provide you with a URL that points to your deployed sample application.

## BUILD-Deploy Microservices Application
In real world, you would have a microservices application to be deployed. Following will walk you through individual commands of Copilot that can deployed individual components of your microservices application, build a release pipeline and showcase how can view logs & health status of your application.

**About the Sample Microservices Application**
This is a reference architecture that shows a containerized Node.js microservices application to using AWS Copilot.

The microservices app was developed from the AWS Sample [here](https://github.com/awslabs/amazon-ecs-nodejs-microservices/tree/master/3-microservices)

The sample has 3 services - users,threads,posts - defined behind an Amazon Application Load Balancer (ALB), and we create rules on the ALB that direct requests that match a specific path to a specific service.
So each service will only serve one particular class of REST object, and nothing else. This will give us some significant advantages in our ability to independently monitor and independently scale each service.


Microservices Application Architecture 
![Sample Application Architecture ](/images/sample-app-architecture.png)

![Architecture - Microservices Deployment with copilot ](/images/Microservices-Architecture.gif)

The typical flow of the commands will be based on above architecture.

1. Create the app 
```shell
git clone https://github.com/gaonkarr/aws-copilot-demo.git && \ 
cd aws-copilot-demo &&                                        \
copilot app init social-media-app
```

2. Create the **TEST** environment in Region of your choice. Following creates it in ap-southeast-2.

```shell
copilot env init --name test --region ap-southeast-2 --default-config
```

3. Create the **PROD** environment in Region of your choice. Following creates it in ap-northeast-1. Notice the --prod flag. This is helpful to identify production environments. It is also automatically considered production when the pipeline is create in step .

```shell
copilot env init --name prod --region ap-northeast-1 --default-config --prod
```

4. Create a Service for "**Users**"

```shell
copilot svc init --name users                       \
--svc-type "Load Balanced Web Service"              \
--dockerfile ./services/users/Dockerfile
```

5. Step 4 creates a manifest file for the service under folder 'copilot/users/manifest.yaml'. In the manifest file, can make any changes to the default configurations for the service. Open this file and add following code to the end. This will tell configure the test environment with 1 task while the prod environment will have 2 tasks. Saved the file before proceeding. 

```YAML
environments:
  test:
    count: 1               # Number of tasks to run for the "test" environment.
  prod:
    count: 2               # Number of tasks to run for the "prod" environment.
```

6. Deploy the "**Users**" service in **TEST** environment

```shell
copilot svc deploy --name users --env test
```

7. Deploy the "**Users**" service in **PRODUCTION** environment

```shell
copilot svc deploy --name users --env prod
```

***Optionally,***
You can repeat steps 4-6 for every new service you wish to deploy. 
Example, repeat following for the "**Threads**" and "**Posts**" services:

```shell
copilot svc init --name threads --svc-type "Load Balanced Web Service" --dockerfile ./services/threads/Dockerfile 
copilot svc deploy --name threads --env test
copilot svc deploy --name threads --env prod
copilot svc show


copilot svc init  --name posts --svc-type "Load Balanced Web Service" --dockerfile ./services/posts/Dockerfile
copilot svc deploy --name posts --env test
copilot svc deploy --name posts --env prod
copilot svc show
```

You can open each service in each environment using the URL provided in the output. Make sure to add __'/api/service-name'__ to view each service individually.

7. Use the show command to check status of your environment deployments

```shell
copilot svc show
copilot env show --name test
copilot env show --name prod
copilot app show
```

![copilot svc show output ](/images/svc-show-output.png)
![copilot env show test output ](/images/env-test-show-output.png)
![copilot env show prod output ](/images/env-prod-show-output.png)
![copilot app show output ](/images/app-show-output.png)

## RELEASE-Deploy Release Pipeline for the Microservices Application

1. Initialize the CodePipeline creation. This will create manifest files - pipeline.yml and buildspec.yml in the copilot folder.
```shell
copilot pipeline init
```

2. Add these to your git repository, commit and push your code. 
```shell
git add copilot/pipeline.yml copilot/buildspec.yml copilot/.workspace && \
git commit -m "Adding pipeline artifacts" &&                             \
git push
```
3. Deploy the copilot pipeline using update command and check the status.
4. 
```shell
copilot pipeline update
copilot pipeline status

```
![copilot pipeline status output ](/images/pipeline-status-output.png)

## OPERATE-Monitor Logs and Status
```shell
copilot svc logs
```
![copilot svc logs output ](/images/svc-logs-output.png)

```shell
copilot svc status
```
![copilot svc status output ](/images/svc-status-output.png)



## Clean up

Remember to delete all the resources once you are done testing.

```shell
copilot app delete
```
![copilot app delete output ](/images/delete-app-output.png)

## References and more

Explore advanced features – addons, storage, sidecars, etc

AWS Copilot CLI repository - https://github.com/aws/copilot-cli/

AWS Copilot CLI documentation - https://aws.github.io/copilot-cli/

Guides and resources - https://aws.github.io/copilot-cli/community/guides/ 

Containers from the Couch videos - https://www.youtube.com/c/ContainersfromtheCouch/search?query=copilot



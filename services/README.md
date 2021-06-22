## ECS Microservices application on AutoScaling Group & Capacity Providers

For the NEW Microservices application, we will henceforth call this NEW_Microservices.

In this sample, we will implement some of the strategies discussed here into a Microservices Nodejs sample application. Lets call it Old (pre-applying some of these strategies) and New(after implementing strategies).

Reference link : 

In this example the microservices application is deployed on ECS.


![Reference architecture of microservices on EC2 Container Service](../images/new-microservice-containers-ecs.png)


## Changes made to the architecture

The sample has 3 services defined behind an Amazon Application Load Balancer (ALB), and we create rules on the ALB that direct requests that match a specific path to a specific service.
So each service will only serve one particular class of REST object, and nothing else. This will give us some significant advantages in our ability to independently monitor and independently scale each service.

__Cluster AutoScaling and Capacity Providers__

ECS Cluster Auto Scaling is a new capability for ECS to manage the scaling of EC2 Auto Scaling Groups. With Cluster Auto Scaling, you can configure ECS to scale your ASG automatically, and just focus on running your tasks. ECS will ensure the ASG scales in and out as needed with no further intervention required. To fill the gap of Managed Autoscaling, where ECS and EC2 instances can communicate scaling effectively, we launched a new feature in ECS called as ECS Capacity Providers. Cluster Auto Scaling relies on ECS capacity providers.

__Use combination of OnDemand with Spot instances__

Spot Instances have same as pay-as-you-go pricing as On-Demand, but at up to 90% off. EC2 can reclaim with a 2 minute warning. Best for stateless or fault tolerant workloads, and that additional scale capacity.
Over the last few years we have made significant changes to the spot pricing models. Each instance family and size, in each availability zone in every region is a separate spot pool, meaning that both price and capacity on each pool fluctuates infrequently and independently from each other; so if you use the same instance type on three different AZs you’re effectively consuming capacity from three different spot pools. 

__Use combination of OnDemand with Spot instances__

__Convert Launch Configuration to Launch Template__

To benefits from above features, Autoscaling group must be launched with a launch template instead of a Launch Configuration. 

## Prerequisites
You will need to have the latest version of the AWS CLI installed and configured before running the deployment script. 
If you need help installing, please follow the link below:

[Installing the AWS CLI ](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)


## Deployment for NEW_Microservices

Replace items written in <> with actual values.

1. Launch an ECS cluster using the Cloudformation template:

   ```
   $ aws cloudformation deploy \
   --template-file infrastructure/ecs.yml \
   --region <region> \
   --stack-name <stack name> \
   --capabilities CAPABILITY_NAMED_IAM \
   --tags Purpose=CostStrategiesDemo Project=NewMicroservices
   ```

2. Deploy the services onto your cluster: 

   ```
   $ ./infrastructure/deploy.sh <region> <stack name>
   ```
   
3. Deploy the CloudWatch dashboard:
   ```
   $ ./infrastructure/deploy-cw-dashboard.sh <region> <stack name>
   ```

Load test this with your favourite load testing application. 
You could also use, [Distributed Load Testing on AWS Solution](https://aws.amazon.com/solutions/implementations/distributed-load-testing-on-aws/) from AWS Solutions Library.

In my load test, I could see below 
![CloudWatch Dashboard screenshot](../images/new-cloudwatch-dashboard.png)

## Load test Compute cost estimate

Region : US-WEST-1.  
20 m4.large instances – On Demand, 20 m4.large instances – Spot. Consider constant use for a month.

20 instances x 0.117 USD On Demand hourly cost x 730 hrs in a month = 1708.00 USD
On-Demand instances (monthly): 1708.20 USD

Unit conversions: Spot discount*: 73/100 = 0.73
1708.20 USD - (1708.20 USD x 0.73) = 461.21 USD
Spot instances (monthly): 461.21 USD

**Total cost** (On Demand + Spot monthly): 2169.21 USD

**That was approx $400 less than previous sample, inspite of 10 additional instances.**

Further cost savings can be done by purchasing Savings plans. For example, the EC2 Instance Savings Plans rate for m4.large in the US West (N. California) for 1 Year term and All Upfront is 0.0851 USD

Reference -> [AWS Pricing Calculator](https://calculator.aws/#/)

Ultimately, our goal is to achieve long term cost optimization by implementing the strategies. For example, the cost explorer would eventually look like :
![Cost Explorer](../images/cost-explorer-cost-optimisation-trend.png)

Possible future upgrade :
![Reference architecture of microservices on EC2 Container Service with Fargate](../images/new-microservice-containers-ecs-fargate.png)


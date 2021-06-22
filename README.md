
# AWS Copilot Sample microservices demonstration


This is a reference architecture that shows a containerized Node.js microservices application to using AWS Copilot.

The microservices app was developed from the AWS Sample [here](https://github.com/awslabs/amazon-ecs-nodejs-microservices/tree/master/3-microservices)
The sample has 3 services defined behind an Amazon Application Load Balancer (ALB), and we create rules on the ALB that direct requests that match a specific path to a specific service.
So each service will only serve one particular class of REST object, and nothing else. This will give us some significant advantages in our ability to independently monitor and independently scale each service.

## Install and Configure AWS Copilot

### Prerequisites
You will need to have the latest version of the AWS CLI installed and configured before running the deployment script. 
If you need help installing, please follow the link below:

[Installing the AWS CLI ](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)



## AWS Copilot Commands :
Deploy the services onto your cluster: 

   ```
   copilot app init social-media-app
copilot env init --name test --region ap-southeast-1 --default-config
copilot env init --name prod --region ap-south-1 --default-config --prod

==Repeat for every service start==
copilot svc init --dockerfile ./services/users/Dockerfile --name 	users --svc-type "Load Balanced Web Service"
copilot svc deploy --name users --env test
copilot svc deploy --name users --env prod
copilot svc show
==Repeat for every service end==

copilot pipeline init
git add copilot/pipeline.yml copilot/buildspec.yml copilot/.workspace && git commit -m "Adding pipeline artifacts" && git push

copilot pipeline update
copilot pipeline status

   ```


## References & More

Explore advanced features – addons, storage, sidecars, etc

AWS Copilot CLI repository - https://github.com/aws/copilot-cli/

AWS Copilot CLI documentation - https://aws.github.io/copilot-cli/

Guides and resources - https://aws.github.io/copilot-cli/community/guides/ 

Containers from the Couch videos - https://www.youtube.com/c/ContainersfromtheCouch/search?query=copilot



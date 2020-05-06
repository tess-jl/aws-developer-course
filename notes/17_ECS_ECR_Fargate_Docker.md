# Section 17: ECS, ECR, and Fargate-Docker in AWS 
-How to use Docker containers in AWS
ECS: 
* Cluster 
* Services
* Tasks 
* Tasks Definition
ECR
Fargate for how ECS can be done in serverless way
https://aws.amazon.com/blogs/compute/building-blocks-of-amazon-ecs/

### What is Docker? 
**Docker** = software dev platform used to deploy apps
* revolutionary b/c of **containers** that can be run on ANY OS 
--> because run same way on any OS, can run an app on: 
* any machine 
* no compatability issues 
* predictable behavior 
* less work 
* easier to maintain and deploy 
* works with any language, any OS, any tech
--> we need to consider how to make a Docker container from our app so that it can be run anywhere 

Means that if we have an OS, e.g. an EC2 instance as a server: 
-can deploy multiple Docker containers, one with Java app, one with Node.js app, one MySQL DB
-even though they're on same machine they don't interact/don't impact each other unless we tell them to 
-run on same server, therefore can scale them!
--> from servers perspective it just runs and removes Docker containers! 

Where are Docker Images stored? 
**images** (what's run on the machine) are stored in Docker repos 
-famous repos: 
1. **Docker Hub** https://hub.docker.com/ --> find images for any OS or tech
1. **Private Amazon ECR (Elastic Container Registry)** 

Hands on
-Docker hub--> can find any images we're interested in, even Amazon stuff! all public

Difference between Docker and a Virtual Machine(VM)? 
* Docker is sort of virtualization tech, not not exactly 
* Docker resources = shared with host, therefore MANY containers on one server 
-**Docker daemon** replaces the **hypervisor** in the underbelly 
* because the daemon is more lightweight can run way more containers than VMs

Getting started with Docker: 
-download Docker 
-create a Docker file (describes how to build a Docker Image), https://www.docker.com/get-started
-build docker image
-run images to make a docker container 
-need to setup pushing docker images to docker hub (for public images) and/or ECR for private images --> can also pull them 

Docker containers need to be managed 
-need container management platform, 3 choices: 
1. **ECS** (amazon's own platform)
1. **Fargate** Amazon's own for Serverless
1. **EKS** = Amazon's managed Kubernetes platform (open source) (Kubernetes = popular container management system)

### ECS Clusters
**ECS Cluters** = logical grouping of EC2 instances 
* instances we launch will run an **ECS Agent** = Docker container --> will register the instance with the ECS cluster
* EC2 instances here = special, don't run plain Amazon Linux 2 AMI, instead use AMI specifically for ECS

Hands on 
ECS console --> create cluster--> (trying to push us to use Fargate) --> use EC2 Linux and Networking API with ASG --> use t2 micro instance type 
-EC2 AMI ID = the same ID as the ECS AMI 
-GUI walks through all services and config --> launch cluster

ECS instance = linked to EC2 instance- it's in specific AZs
* CPU available = the VCPU have on machine, will be shared between different containers --> same princple for memory 

EC2 console --> ASGs --> the cluster came with ASG
* launch configuration = special for ECS, has User Data --> therefore when instance starts it knows to register to this cluster! 
* has SG

SSH into the machine --> the ecs.config file has code that shows how it knows which cluster to register to --> how does it register? Docker does it --> the agent is registering our instance into the ECS service!
* can also look at the Docker logs via CLI 

### ECS Task Definition 
**ECS Task Definition** = metadata (JSON) to tell ECS how to run a Docker container, includes: 
* Image Name 
* Port Binding for Container and Host 
* Memory and CPU required 
* Env vars 
* Networking info 

actual server, e.g. Apache, has port 80 but it is mapped to a host port of 8080 --> why? b/c any traffic to 8080 will be mapped to 80 now 

Hands On 
create new task def --> task role = important (if exam asks why container cant talk to S3, etc. it's because it's missing a role) --> task execution role --> choose memory and CPU --> choose image (e.g. httpd apache for us)
* port mappings = host = 8080, check what's exposed within the container 
--> have container name httpd from image etc. --> click create 
-tag 1 

### ECS Service
-to run task need to define an **ECS Service** = will define how many tasks should run and how they should run 
* also ensure # of tasks desired is running across the fleet of EC2 instances
* can be linked to any LB if needed 

Hands On 
cluster --> create a service --> launch type, task def
* daemon vs replica for service type --> daemon is when want to run something across all instances
* configure network --> LB --> ASG --> review
view service --> service will launch the httpd task 

cluster --> can see that task is running 
```
docker ps
```
can see what has happened 
-scale service to run 2 tasks --> update service 
**can only have 1 task/ECS instance!** so instead need to scale our cluster --> create another t2micro that will appear when scaling up 

### ECS Service with LB 
-change task def but DO NOT specify a host port (only specify port for container), therefore it becomes random --> able to work with a LB 
- with random port #s hard to direct traffic! but **ALB can do it via dynamic port forwarding**

Hands On 
-leave host port as empty in def (port 0, means undefined, random)
-update service to the updated task def
-can see via curl in CLI that two instances / apps are running but need to be LB'd 
-cannot add LB as go along --> needs to be set on service creation 

create new service --> ALB (for **dynamic host port mapping** with route dynamically, very smart) -> create new LB for this and then create new IAM role for security group 

to delete a service --> update for 0 number of tasks and then delete 

get DNS name aka URL for LB --> put in browser to show how ALB redirecting to instances via dynamic port mapping 

### ECR Part 1
-so far only been using the public Docker images but now want to use ECR, a private repo, therefore we'll be able to build and push our own image 
**ECR(Elastic Container Registry)** = private Docker image repo 
* access controlled via IAM perms
* need to run push/pull commands to Docker repo --> first use 
```
$(aws ecr get-login --no-include-email --region us-east-1)
```
gives a huge docker login --> generates a temp Docker login against an ECR repo --> when has $() syntax it WILL log in 

then:
```
docker push [ECR_URL]
```
or pull with same syntax 

Hands On 
-see Dockerfile in /code 
-bootstrap will run to pick up some meta data 
-in directory with these files
```
docker build -t [nameOfImage] .
```
--> now have built out first Docker Image 
--> NOW need to push to ECS first! 
-run command to tag (rename) image 
--> docker push command will push it into the ECR --> in repo can see the tag, Image URL, etc. 
* to download this image run same exact login command and then do the docker pull command with right image name and right tags! 

### ECR Part 2
-need to update service --> create new task def that will represent the service --> update it to the new image name from ECR
* now the image will not be pulled from Docker hub but from the ECR
--> update the service for the new task 

-can look at the ALB in the LB console and see the target groups --> see how some tasks are draining, a rolling restart

-get the DNS from the LB --> in browser --> can see if works! 
-can see if refresh how LB is redirect to different tasks 

--> EC2 instances are able to pull the data from ECR b/c has right IAM perms (if look at policy can see it's able to read all resources from ECR, why it's able to pull Docker image from ECR, it's quite privileged!)

### Fargate 
-before fargate every time launch ECS cluster have to create our own EC2 instances, if need to scale we need to add instances and scale services, manage our own infastructure!
NOW = **Fargate** = serverless way to do same thing
* don't have to provision EC2 instances, **just have to create a task definition and AWS will run containers for me**
* **to scale just increase the number of tasks!**
--> revolutionary --> the way to run Docker Image in the cloud 

Hands On 
-create a new cluster --> networking only --> powered by Fargate 
-current task def is not compatible with Fargate --> create new task def with launch type Fargate --> select task configs (RAM and CPU) --> add container  and AWS takes care of rest 

-create a Service for this --> configs --> config network --> add it to ALB --> create service 
-NO EC2 instance within Fargate service!! will do this behind the scenes 

-go to DNS in browser --> see the DockerName that is running **ecs-fargate-...** and the **"NetworkMode" : "awspc"** also shows it's Fargate 

### ECS and X-Ray 


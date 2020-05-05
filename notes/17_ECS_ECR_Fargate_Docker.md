# Section 17: ECS, ECR, and Fargate-Docker in AWS 
-How to use Docker containers in AWS
ECS: 
* Cluster 
* Services
* Tasks 
* Tasks Definition
ECR
Fargate for how ECS can be done in serverless way

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

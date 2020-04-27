# Section 13: AWS Serverless: Lambda 
-new paradigm 
-Lambda = one of the most widely-used and popular, revolutionized how deploy, scale, make apps 

### Serverless Intro 
**serverless** = new paradigm in which developers don't manage servers --> developers deploy code.. functions! 
* initially serverless = **FaaS** (Function as a service) 
* now, serverless = anything that doesn't require servers 
* most of the time in AWS serverless = lambda
* **serverless != no servers... just that don't manage, provision, see them**

Serverless in AWS 
* Lambda and Step Functions 
* DynamoDB
* Cognito 
* API Gateway 
* S3
* SNS and SQS
* Kinesis 
* Aurora serverless (RDS but without creating a server)

AWS takes care of server architecture management for all of them :) 

### Lambda Overview
with EC2 instances provisioned in the cloud --> limited by RAM and CPU, continuously running, BUT for scaling need to step up intervention, need to add/remove servers i.e. **managed the servers**
VS.
Lambda --> **virtual functions** therefore no servers to manage but **short executions**, run on demand, scale within milliseconds, scaling automated, by default meant to scale to load that is happenign 

Benefits: 
* pricing easy --> **pay per req and compute time**
* free tier of 1mill req, 400,000 GBs of compute time 
* integrated with whole AWS stack 
* integrated with many programming languages 
* easy monitoring via CloudWatch 
* eay to get more resouces/function (up to 3GB of RAM) --> increasing RAM also increases CPU and network 

languages and versions
* Node.js
* Python 
* Java
* C# (.NET Core)
* Golang 
* C#/Powershell 

Main Services integrated with Lambda
* API Gateway 
* Kinesis 
* DynamoDB
* AWS S3
* IoT
* Cloudwatch Events and Logs 
* SNS
* SQS
* Cognito

Example: serverless Thumbnail creation 
-scales infinitely!!! 

Example: serverless cron job 
-usually cron jobs run on EC2 machines running Linux (server) BUT this is serverless!

Pricing
-see docs
-**pay per calls** 
-**0.20/1 million calls after 1st free 1mill**
-**pay per duration** --> increment of 100ms
* 400,000 GB second (i.e. 400,000 seconds if function is 1GB RAM)--> proportionally scales
* beyond this, $1/600,000 GB-seconds
-generally very cheap, therefore popular

### Lambda Hands On 1





REVIEW
-CPU, network, RAM 



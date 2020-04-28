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
-no charge when code not running
-integrate with many different backends 
-more we send events the more we have to pay (cost linked to invocations and how long invocations last)

to create function --> 3 options:
1. Scratch 
1. blueprint 
1. browse app repository 
-doing hello world python --> choose an execution role (IAM role) --> select basic role for lambda --> create function 

can test function --> configure test event --> test --> result:succeeded shows logs with outputs from function, duration of function, billing, resources configed, memory used, etc. 
-can also view logs in CloudWatch log group

-also provides a UI with viz of services
-can also click link to IAM role for this lambda function

### Lambda Configuration
-**timeout** for how long function will run before failing --> **default = 3s, max=15 min**
-**env vars**
-can allocate memory for lambda function (128M to 3G)
-can deploy within a VPC, assign security groups 
-IAM role must be attached to Lambda function 

Hands On 
using the Designer UI can see what the lambda funciton interacts with
* able to add things that can trigger a lambda function (i.e. many diff services)

for function code 
-can edit inline, upload a .zip, or upload from S3
-select the runtime (languages with version)

Env vars 
-key/value pair modularized from code 
-allow for a plug and play 
-can encrypt them 

Tags 
Descripton 
Memory (more memory, more CPU, higher cost)
Timeout (how long function expected to run)
Network --> allows for selection of a VPC to run in --> if no VPC will have better performance --> if VPC might be slower to start, also need to select subnets with VPC
Auditing 
etc. 

if function runs beyond timeout --> see that execution result: failed --> error message declarative as to why
-set timeout accordingly

### Lambda Concurrency, Throttling, and DLQ
-**concurrency** = **up to 1000 executions** at once in region --> therefore can auto scale up to this many functions running at the same time
* can be increased through **ticket**
* can set **reserved concurrency** at the function level (e.g. limit to 100 functions)
* each invocation over the concurrency limit will trigger a **throttle** 

Throttle behavior 
-if function is sync --> **ThrottleError-429** 
* if fails responsible for retrying yourself
-if function is async --> will **auto retry**, if fails too many times will go to **Dead Letter Queue (DLQ)**
* if fails once, auto retried twice 
* if still doesn't work then will go to DLQ if we set it up
* **DLQ can be either SNS topic or SQS Queue** with **correct IAM execution role**
* original event payload will be sent to DLQ 
* easy way to debug 

Hands On 
-debugging and error handling have been replaced by X-Ray 
-update IAM perm
-can throttle test it --> see "Rate Exceeded" as the result of the test!

### Lambda Logging, Monitoring, Tracing
-all logs go to CloudWatch 
* need right IAM role 
* Lambda metrics --> in CloudWatch metrics 
* **need to make sure Lambda function has an execution role with an IAM policy that allows writes to CloudWatch** 
-can enable integration with X-Ray 
* can trace Lambda with X-Ray 
* enable in Lambda config (runs daemon for us)
* use AWS SDK in Code 
* **ensure Lambda function has correct IAM execution role** 

Hands On 
monitoring tab in lambda --> cloudwatch metrics i.e. how many times function invoked, duration (used to set appropriate timeout!), errors, availability, throttles, DL errors, concurrent executions (used to set appropriate concurrency rules)

x-ray --> can enable active tracing in Debugging and error handling area --> lambra role changed to have some x-ray permissions --> can go to IAM to check the perm 
--> can see the service map in x-ray to look at the lambda service

### Lambda Limits





REVIEW
-CPU, network, RAM 
-Image 




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
Lambda --> **virtual functions** therefore no servers to manage but **short executions**, run on demand, scale within milliseconds, scaling automated, by default meant to scale to load that is happening 

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
-can deploy within a VPC (has it's own network IP address), assign security groups 
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
* can be increased through **ticket** --> need to submit a special ticket to AWS
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
**execution limits** 
* memory allocation 128-3008 MB (64MB increments)
* max execution time 15 min, default 3s
* Disk capacity in the function container = 512MB(in /tmp directory, can write this much data here)
* concurrency limit = 1000

**deployment limits**
* lambda function deployment size (compressed .zip) 50 MB max 
* uncompressed code and dependencies= 250MB max
* if want to go over 250MB can load files or dependency at startup and write to /tmp directory 
* size of env vars = 4KB max 

### Lambda Versions and Aliases
**versions**
* when work on function, working on the version called **$LATEST** (**mutable**)
* when ready to publish (snapshot latest and create a new version)--> create version (e.g. v1), next publish v2, etc... --> once published = **immutable** --> these versions = code AND configuration (env vars, timeouts, etc) all immutable
* **each version gets own Amazon Resource Name (ARN)** --> therefore able to access a specific version with ARN
* create versions to have dev, test, prod envs 

**aliases** = better for setting up diff dev, test, prod envs
* aliases = **pointers to Lambda function versions (one or two)** --> MUTABLE!
* can define dev, test, prod aliases and have them point to diff lambda versions 
* users interact with the dev alias, for example 
* great because can do a blue/green deployment using aliases and assigning weights to lambda functions versions (can test with traffic) but from user standpoint endpoints are the same (routing to actual versions is just different)

Hands On 
action--> publish --> snapshot --> version 1 --> can talk to cloudwatch logs, SQSm etc. IMMUTABLE 

want want to expose to user is dev, test, prod endpints --> need to create aliases 
-create a dev --> now can manage lifecyle of DEV 
-create prod pointing to v1 --> but we want it to pont to v2! want to shift traffic between two versions with the weighted traffic to see how it works 
-alias can be managed independently from versions

### Lambda and External Dependencies
popular exam q: how do we bundle external dependencies with lambda function? 
* if lambda func depends on external libraries (X-ray SDK, Database clients, etc.) --> **need to install packages in codebase and .zip together** 
* done with npm i --> node_modules dir for Node.js, pip --target options for Python, etc.
* with .zip --> upload striaght into lambda if **< 50MB**, otherwise S3 first and reference it
* Native libraries OK but they need to be compiled on Amazon Linux first

Hands On 
see the require('aws-xrat-sdk-core') require('aws-sdk') and  library installation in index.js 
npm i aws-xray-sdk
```
zip -r function.zip
```
to zip code and dependencies 

--> in lambda console: author a new function, choose Node.js runtime, create a new role
NOTE editing code inline on GUI does not allow for adding dependencies 
--> save function (i.e. code uploaded)
go to IAM role --> attach more policies, give read only access to S3, give write access to x-ray and of course the basic execution role for lambda (auto)
--> the designer on lambda should reflect these permissions 
--> ALSO make sure to test function --> works and also look at x-ray to prove that the external library worked 

### Lambda and CloudFormation 
-**must store lambda .zip in S3**
-refer to the S3 zip location in CloudFormation code (regions need to be same)
-done in the codeblock labeled **Code:** in template --> S3Key key has value that is the zip file and S3Bucket key has value that is the name of the bucket --> BOTH ARE PARAMS?? 

Hands On 
cloudformation template with params --> Fn::GetAtt used and in the Code: subblock in properties blocl see that each key is a refrence to a param

-need to create a new stack in cloudformation and upload the template --> stackname is LambdaThroughCloudFormation --> auto shows params and then enter values of params --> tick box to creat IAM role --> create stack and cloudFormation will provision lambda function for us!
-so now can go to lambda and see the function from the S3 bucket .zip that was provisioned

### Lambda /tmp directory 
the /tmp space used when: 
* function needs to download big file to work 
* function **needs disk space to perform operations in some way**

-max size 512MB 
-**content remains even if the execution contect is frozen** --> provides transient cache that can be used for multiple invocations (helpful for troubleshooting)
-**temporary, not persistent** --> use S3 if want persistent data

### Lambda best practices
**perform heavy duty work outside of function handler**
* e.g. connect to DBs outside of function handler otherwise everytime function handler invoked will connect to DB
-same reasoning for: 
* not initializing AWS SDK in function handler
* not pulling in dependecies (instead use **execution context**)
* just put work above the actual function in the file --> done outside of function

**use env vars for**
* DB connection strings, S3 buckets, etc.
* anything sensitive like passwords, can also be encrypted using KMS 
(remember 4KB size max)

**minimize deployment package size to its runtime necessities** 
* break down function into several functions if too big  
* recall lambda limits (50MB compressed, 250 not compressed)

**never use recursive code**

**don't put lambda in VPC unless have to** otherwise will take long to initialize 

Hands On
-see how to alter code to move heavy duty work

### Lambda@Edge
-CloudFront (on Edge) = service used to deploy onto **CDN (content delivery network)** so that functions run globally
* what if wanted to run global AWS lambda alongside this? 
* or what if wanted to implement request filtering before reaching app? 

--> use Lambda@Edge = means don't deploy functions in a region but in every single cloudFront edge location aka globally 
-therefore: 
* more responsive apps 
* don't need to manage servers
* customize CDN content 
* pay for what we use 

Lambda changes CloudFront req and res (4 types of ways):
1. **viewer req** = after CloudFront receives a req from viewer 
1. **origin req** = before CloudFront forwards req to origin 
1. **origin res** = after CloudFront recevies the res from the origin 
1. **viewer res** = before CloudFront forwards the res to the viewer 

--> LAMBDA LIKE A MIDDLEWARE?

-can also not even go to origin, Lambda itself could be the "origin" and return a res right away (i.e. generate responses to viewers without ever sending the request to the origin) --> therefore possible to run global response app with CloudFront and Lambda@Edge serverless

Lambda@Edge
* website security and privacy 
* dynamic web app at the Edge 
* search engine optimization (SEO)
* intelligently route across origins and data centers
* bot mitigation at Edge (by filtering bad req)
* RT image transformation 
* A/B testing 
* user auth
* user tracking and analytics



REVIEW
-CPU, network, RAM 
-Image 
-Drivers (Node.js drivers)
-why aliases used instead of versions? 
-done in the codeblock labeled **Code:** in template --> S3Key key has value that is the zip file and S3Bucket key has value that is the name of the bucket --> BOTH ARE PARAMS?? 

--> LAMBDA LIKE A MIDDLEWARE?
Lambda@Edge
* website security and privacy 
* dynamic web app at the Edge 
* search engine optimization (SEO)
* intelligently route across origins and data centers
* bot mitigation at Edge (by filtering bad req)
* RT image transformation 
* A/B testing 
* user auth
* user tracking and analytics

Lambda
-can deploy within a VPC (has it's own network IP address), assign security groups 

-when do things get removed from the /tmp directory
-lambda will re-use function whenver possible --> if functions being called multiple time in a row or in parallel

lambda file vs handler --> what is considered the function 

**CDN (content delivery network)**

CloudWatch Events --> can configure lambda to be invoked in response to an event --> WATCH VIDEO!!
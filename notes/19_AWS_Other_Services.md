# Section 19: AWS Other Services
-overview of services 
-exam will ask very basic level questions about CloudFront, Step Functions and SWF, SES, ACM

### Section Overview
-remember the general ideas of CloudFront, Step Functions and SWF, SES, ACM, no hands on or in depth

### AWS CloudFront
**CloudFront** = a **Content Delivery Network (CDN)**, usually related to S3
* improves read performanced because content is cached at the Edge
* Edge locations = 136 point of presence locations globally--> therefore content delivered to these 136 places globally by this CDN 
* popular with S3 but works with EC2 and LBs
* helps protect against network attacks 
* can have SSL encryption (HTTPS) at edge using ACM
* can use SSL encryption(HTTPS) to talk to my apps --> fully secure, fully encrypted service
* supports RTMP protocol for videos/media


e.g. with bucket on other side of world instead of user talking directly to bucket (will take a long time) user will talk to edge location and then edge location will talk to the bucket, cache the data, and use cache when other users req stuff --> therefore S3 bucket will be cached across the world

### Step Functions and SWF
**Step Functions** = serverless visual workflow for orchestrating Lambda functions 
* if have a lot of lambda functions that need to work together can use step functions to do it
* represent the flow as a **JSON state machine**
* can use the sequences of lambda functions, conditions, timeouts, errorhandling etc. of the functions to do orchestration 
* can integrate with EC2, ECS, On-premise servers, API Gateway etc. but it's less popular 
* max execution = 1 yr 
* possible to implement human-approval feature 
* use cases: Order Fulfillment, Data processing, web apps, any **workflow**

**Simple Workflow (SWF) Service** = for coordinating work amongst apps 
* code = on EC2 (not serverless)
* 1 year max runtime 
* concept of **activity step** and **decision step** 
* built-in human intervention step 
* similar to step functions --> e.g. use for order fulfillment from web to warehouse to delivery 
* OLDER than Step Functions, so only use case for SWF now is: **if need external signals to intervene in process** or if **need child processes that return values to parent processes** 

### AWS SES
**Simple Email Service (SES)** = to send or receive emails to people
-send using either: 
* SMTP interface
* AWS SDK 
-receive email integrates with: 
* S3
* SNS
* Lambda
-integrated with IAM for allowing to send emails 

### Summary of Databases (OLTP, OLAP, NOSQL, CACHE)
**RDS** = for relational databases, transaction processing (OLTP), SQL
* PostgreSQL, MySQL, Oracle.. 
* Aurora and Aurora Serverless
* provisioned

**DynamoDB** = NoSQL DB 
* Managed, Key Value, Document store 
* serverless (we just provision capacity)

**ElastiCache** = in-memory DB 
* Redis or Memchached
* great for storing state in memory 
* caching 

**RedShift** = OLAP, analytic processing 
* **Data warehousing / Data lake**
* analytics queries 

**Neptune** = graph database 
* newer

**Database Migration Service (DMS)** = quickly enable us to load data on to any of these DBs

### Amazon Certificate Manager (ACM)
**Amazon Certificate Manager (ACM)** = used to host public SSL certificates in AWS, can either: 
* buy own certificates and upload them via CLI 
* have ACM provision and renew public SSL certs for me for free 
ACM loads SSL certs onto: 
* LBs (including LBs created by EB)
* CloudFront distributions 
* APIs on API Gateways

--> SSL certs are annoying to manage manually so ACM is great way to do it 
**ACM helps with SSL termination on the service with the SSL cert** therefore further reqs that are happening to other services internally on the backend can be done on HTTP (HTTPS no longer)
* therefore less CPU cost on EC2s because don't have to encrypt/decrypt payload and don't have to manage SSL certs for EC2s

Hands on 
open ACM console --> req public cert from amazon --> add domain name --> choose DNS validation to validate that we do own domain name (in this case it's in Route53) --> need to create a CNAME record in DNS config for validation to work
-just have record created in Route 53 for us --> create the CNAME record for us
-wait for validation --> cert issued

use certificate in EB--> create an env that leverages that cert --> add the SSL cert in custom config --> create EB app 
-LB should have our SSL cert to access our env from there 
-created but need to have CNAME record to create the right domain name for our app --> route 53 --> go to domain --> create CNAME record NOW check in browser and it has right domain name with HTTPS

-verify that cert in use by going to ACM --> can also go to ELB console --> look at the listeners 






REVIEW 
-RTMP protocol
-state machine
-use cases: Order Fulfillment, Data processing, web apps, any **workflow**
-concept of **activity step** and **decision step** 

-OLDER than Step Functions, so only use case for SWF now is: **if need external signals to intervene in process** or if **need child processes that return values to parent processes** ??? 

-SMTP interface

OLTP
Aurora and Aurora Serverless --> thought Aurora always serverless? 
OLAP = analytic processsing --> Data warehousing / Data lake

SSL termination on a load balancer --> what is it? 

SSL policies 


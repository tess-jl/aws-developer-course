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



REVIEW 
-RTMP protocol
-state machine
-use cases: Order Fulfillment, Data processing, web apps, any **workflow**
-concept of **activity step** and **decision step** 

-OLDER than Step Functions, so only use case for SWF now is: **if need external signals to intervene in process** or if **need child processes that return values to parent processes** ??? 

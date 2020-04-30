# Section 15: AWS Serverless: API Gateway and Cognito 
-what if want to expose functions to the world? 
-what if want ppl to access our app using a REST API? --> API Gateway will be able to spread our app and expose it as a RESTful API 
-will also be able to do user auth etc. on top of it with Cognito 

### API Gateway Overview
-used to Build, Deploy, and Manage APIs 
-APIs = interfaces can expose to people for them to get data 
**API Gateway** = for controlling over what the client can and can't do, business logic handled by Lambda 
* no infastructure to manage
* handle versioning, diff envs, security (from from lambda and instead to security here)
* create API keys, handle req throttling 
* integrated with **Swagger** and **Open API import** to quickly define APIs, export them as SDKs, etc. 
* transform and validate req and res
* generate SDK and API specs 
* cache API responses (i.e. limit load on lambda function)

API Gateway Integrations 
Outside of VPC: 
* Lambda (most popular/powerful)
* any AWS service (EC2, LBs, etc.)
* proxy from API Gateway to external and publicly-accessible HTTP endpoints
Inside of VPC: 
* Lambda 
* EC2 endpoints

### API Gateway Basics Hands On 
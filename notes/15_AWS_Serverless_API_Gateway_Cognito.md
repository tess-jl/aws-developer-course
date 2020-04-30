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
gives an example API but we're creating a new API 
MyFirstAPI --> resources, stages, authorizers, gateway reponses, models, resource policy, docs, settings 
-create a CRUD method with any verb
-can choose integration type -> whether a lambda, an HTTP, a mock, etc.
-go to lambda service to create a lambda function for it--> api-example-root-get TestEvent function
-can test! tells us latency, status, body etc. 
-create another lambda function for the /houses endpoint --> api-example-houses-get
--> in 1 API have integrated 2 lambda functions 
**resources** = the path (e.g. /houses) in API
**methods** = CRUD

### API Gateway Stages and Deployment
common misconception = **when make changes in API gateway it does not mean they're effective right away!! NEED TO DEPLOY!**
--> changes are deployed to **stages**, can have as many as we want (whatever naming we want, dev, test, prod, etc.)
* each stage independent from one another, own configs
* stages can be rolled back (history of deployments kept)

Why use stages? 
-can change the gateway behavior depending on variables
-b/c can have **stage vars** = like env vars for Gateway --> used to change config 
Can use Stage Variables for: 
* lambda function ARN
* HTTP endpoint 
* parameter mapping templates, etc.
Use cases: 
* config HTTP endpoints that stages talk to (dev, test, prod, etc.)
* pass config params down to lambda function through mapping templates so that function knows if it's in dev, test, prod
**stage vars are passed to the "context" object in lambda** --> therefore have ways to get these vars back in lambda 

How do we use stage variables? 
-common pattern = with **lambda aliases**
* API Gateway will auto invoke the right lambda function 

**Canary Deployment** = a test deployment, can be done at any stage but usually prod
* choose the % of traffic the canary channel receives, small percentage goes to new version/channel 
* metrics and logs for channels are separate 
* ~ blue/green deployment with Lambda and Gateway

### API Gateway Stages and Deployment Hands On
actions--> deploy API to stage called "dev" --> now whole API lives under the dev stage in UI 
* NOW we have a URL we can use to invoke our stage! --> can see the res from lambda! 
* shows how we've built and deployed a RESTful API already 

-add stage var named alias with value dev 
-can export API as a lot of things 
-deployment hisotry

### API Gateway Mappings
**mapping templates** = for modifying reqs or res, can: 
* rename params 
* modify body content 
* add headers
* map JSON to XML for sending to backend or back to client 
-uses **Velocity Template Language (VTL)** (uses for loop, if statements, etc.)
* can filter output results (remove unnecessary data)

--> can edit in **integration req or res** 

Hands On 
if wanted Dev API not to return JSON but to return XML 
resources --> select endpoint --> change res for example --> integration res edit 
-choose mapping template --> choose empty model --> save
-do a test and see that the res body is a XML 
NOW need to action --> deploy API to actually see the changes 

### API Gateway Swagger
**Open API fka Swagger** = common way of defining RESTful APIs using API definition as code --> written in either YAML or JSON 
* can import OpenAPI 3.0 spec to API Gateway and it will just generate an API for us! 
when this happens aut get: 
* methods
* reqs
* integration req 
* res
* **AWS extensions for API Gateway** --> way to set up every option in the gateway with the extension 
-can also export a current API from Gateway using OpenAPI spec
-can generate SDK for our apps 

Hands On
stages --> dev--> export tab --> Swagger --> get whole YAML file that represents our API therefore our **API as code** 

create new API --> example API uses a Swagger File for API called PetStore API 
-create a stage for this --> deploy API to dev stage

### API Gateway Caching
**caching API res** = reducing number of calls to backend, therefore reduced latency for many calls 
* default TTL = 300s (5min), min=0s, max=3600s (1hr)
* 1 cache/stage 
* can encrypt cache 
* size = 0.5GB-237GB 
* possible to override cache settings for specific methods, therefore control over what gets cached 
* able to flush entire cache (invalidate it) immediately via console or API call 

**can clients invalidate aka clear cache? YES, if they're authorized to do it via IAM** via header **Cache-Control: max-age=0**

Hands On
go to stage --> enable cache --> capcity by increments --> set time, etc. 
-**per-key invalidation** by clients via header **Cache-Control: max-age=0** requires auth 

### API Gateway Monitoring 
-can have Logging, Monitoring, and Tracing 
-integrated with **CloudWatch Logs**
* need to **enable CloudWatch logging at stage level (via Log Level)**
* can override settings on API basis (e.g. ERROR, DEBUG, INFO logs)
* log contains info about req/res body --> helpful for debugging 
**CloudWatch Metrics** 
* metrics/stage
* can enable detailed metrics 
**X-Ray**
* enable tracing to get extra info about reqs to API Gateway

API Gateway + X-Ray + Lambda = full picture

Hands On 
stage --> logs/tracing tab 
-if save it won't work! **need to provide CloudWatch log rol ARN for this gateway** --> go to IAM, create new role, copy ARN and put in gateway console
-invoke URL from stage a few times--> can now see logs --> cloudwatch console log streams for this --> can see the res body before transformation 

-can also go to x-ray and see the service map--> talks to gateway and the endpoint (this is the current tracing)
--> can see how each lambda funciton calls things --> whole vis of how things are working and can see patterns

### API Gateway Others
**CORS** = must be enable when you recive API calls from another domain 
-need to enable **OPTIONS pre-flight req with:**
1. **Access-Control-Allow-Methods**
1. **Access-Control-Allow-Headers**
1. **Access-Control-Allow-Origin**
-can enable CORS via API Gateway console

Hands On 
action --> enable CORS

can set **Usage Plans** and **API Keys**
-if have customers and want to limit API usage --> usage plan for throttling  
* throttling = setting over capacity and burst capacity 
* quotas = API reqs / day / week or / month
* associate with whatever API stage we want

for each usage plan need 1 API Key 
* 1/customer 
* able to track usage/key

### API Gateway Security 


REVIEW
-can the gateway be thought of as a router? 
-Why backslash in the URL for API Gateway deployed?? 
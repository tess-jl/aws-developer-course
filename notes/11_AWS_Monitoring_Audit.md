# Section 11: AWS Monitoring and Audit: CloudWatch, X-Ray, and CloudTrail 
-monitoring app --> tracing, audit etc. enabled!

### Monitoring Overview
-users don't care how app deployed --> users only care that app works 
* latency 
* outage
-want to internally monitor
-look at performance and cost 
-trend (scaling patterns)
-learning for improvement 

AWS has 3 ways to do this: 
via **CloudWatch** 
* **metrics** --> collect and track 
* **logs** --> collect, monitor, analyze, store file logs 
* **Events** --> send notifs when things happen 
* **alarms** --> react in RT to metrics/events 

via **X-Ray** 
* troubleshoot app performance and errors 
* distributed tracing of microservices (can trace API call through tech stack, e.g.)

via **CloudTrail** 
* internal monitoring of API calls being made 
* audit changes to resources by users

### CloudWatch Metrics
-metrics for all services
- **metric** = var to monitor (e.g. CPUUtilization)
* belong to **namespaces**
* **Dimension** = attribute of a metric 
* up to 10 dimensions/ metric 
* metrics have timestamps 
-can use dashboards to see them 

EC2 detailed monitoring 
-default = metrics every 5 min 
-extra cost = DETAILED metrics, < 5 min --> therefore more responsive ASG 
-free = 10 detailed metrics 
-EC2 memory usage not default metric --> must be pushed from inside EC2 instance as a custom metric 

Custom Metrics 
-can use dimensions to segment metrics 
* Instance.id
* Environment.name
-metric resolution --> standard = 1 min 
* < 1 min, up to 1 second = **high resolution** --> API call to enable = **StorageResolution**, higher cost
-**to send metric to cloudwatch --> API call to PutMetricData**
-use exponential backoff 

Hands On 
-CloudWatch console--> metrics --> data vis
-**widget** --> can name graph, can share it, add to dash 
-should know basic EC2, basic EBS

detailed monitoring--> EC2 console--> ASGs --> monitoring tab --> can "Enable Group Metrics Collection"
--> click on instance --> monitoring tab --> "Enable Detailed Monitoring" 

### CloudWatch Alarms 
**alarms** = used to trigger notifs for any metric 
-can be attached to ASG, EC2 actions, SNS notifs
-customizable 
-states: 
* **OK** --> no alarm
* **INSUFFICIENT_DATA** --> unsure if OK or ALARM 
* **ALARM** --> alarm in place, actions happening/happened
-period of alarm (see Alarm preview when creating alarm) --> time (s) to evaluate metric
-high res custom metrics --> either 10 or 30 s

Hands On 
-CloudWatch dash --> Alarms portion
-e.g. alarm for scaling down ASG, when look at scaling policy we see how the alarm is related 
-when creating alarm can control if email is sent

### CloudWatch Logs 
-apps can send logs to CloudWatch via **SDK**
-collections from: 
* EB (from app)
* ECS (from containers)
* Lambda (from function logs)
* VPC flow logs (VPC-specific)
* API Gateway
* CloudTrail based on filter
* CloudWatch Log Agents (on ANY machines, EC2, on-premise..)
* Route 53 (DNS queries)
-**logs can go**:
* batch exporter to S3 (archival)
* stream to ElasticSearch cluster for further analytics (opensource tech)

Logs
-can use filter expressions 
-**storage architecture** 
* **log groups** = arbitrary name, representing app usually 
* **Log streams** = instances in app, log files, containers 
-can define log expiration policies
-can use CLI to watch logs 
-**need to make sure IAM perms correct to send logs!**
-**security - encyrption of logs using KMS at Group level!**

Hands On 
-Cloudwatch--> logs --> log from codebuild 
-can click on Expire Events After value in column to set retention policy for a Log Group 

-in log group can see log stream --> can even search/filter to see specific logs 

-EB --> can **set Health Event Streaming to cloudWatch logs** is important 
-creates cloudwatch log groups for our application 
-can see these logs in the cloudwatch logs console 

### CloudWatch Events
-can define an event aka a scheduled **cron job**
-can also have an **event pattern** --> rules to react to a service doing something (e.g. code pipeline state changes)
-can do **triggers to Lambda functions, SQS/SNS/Kinesis messages**
-events creates a small JSON doc as output to give info about the change

Hands On 
cloudwatch --> events --> **rules**
-targets = most things we would want 
-event pattern --> list of services --> choose code pipeline --> anytime code pipeline fails --> add target SNS with Alarm triggered and create rule 

-some rules auto created

### X-Ray Overview
-for debugging in production, traditional way is arduous, hard to centralize 
-**X-ray** = visual analysis of our apps 

-trace exactly what happens when we talk to one service on the cloud 

-can identify bottlenecks
-understand dependencies 
-pinpoint service issues 
-review req behavior 
-find errors and expections 
-where throttled
-meeting time SLA 
-identify users impacted 

compatibility 
-Lambda
-EB
-ECS
-ELB 
-API Gateway 
-EC2 (or any app server, even on-premis)

How does it work? 
-leverages **tracing** = end to end way to follow a req
-each component dealing with req adds own **trace**
* each trace made of **segments** --> each segment made of **subsegments** 
* **annotations** can be added to traces to provide extra info 
-together --> able to trace every req or sample req

How to enable? 
-(e.g. Java, Python, Go, Node.js, .NET) **code MUST import x-ray SDK** (a tiny bit of code modification)
-SDK will then capture: 
* calls to AWS
* HTTP/HTTPS req 
* DB calls (mySQL, PostgresSQL, DynamoDB)
* Queue calls (SQS)
-Install X-ray daemon or enable X-ray AWS integration 
* **x-ray daemon** = little program that works as low level UDP packet interceptor, sends batch every 1s to x-ray 
* if using Lambda or other AWS services that already run the X-Ray daemon for me 
* **each app must have IAM rights to write data to x-ray** 

--> if x-ray works locally but not on EC2--> b/c of IAM rights for x-ray daemon

What does it do?
-collects data from services 
-service map is computed from all the segments and traces 
-visually-oriented

Troubleshooting
-X-ray not working on EC2 --> IAM role and/or need to have instance running X-ray daemon 
-X-ray not working on Lambda --> ensure it has IAM execution role with the proper policy (**AWAX-RayWriteOnlyAccess**) AND that X-ray vode is imported in code 

### X-Ray Hands On 
x-ray console --> US east N virgina region
-X-ray SDK --> does not send directly to X-ray service INSTEAD it sends to daemon, which uploads in batches (avoids throttling)

daemon can be enabled on: 
* EC2 (linux)
* EC2 (windows)
* ECS
* EB (via **ebextensions/xray-daemon.config**)
* Lambda

running a sample app: 
-via cloudformation template --> with roles, policies, EB app, etc. 
-many resources created
-go to outputs --> EB URL --> go to URL in browser --> see welcome to X-ray app, each min there is a duplicate sign up, should trigger an error 
- **service map** in x-ray computed --> see visual of what is happening
-click on one thing and get extra viz 
-can use console to drill down into traces--> click on trace to see how FE makes req --> response 400 from dynamoDB, etc. 

-distributed traces = very cool 

## X-ray additional exam tips 

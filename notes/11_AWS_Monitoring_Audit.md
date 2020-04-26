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
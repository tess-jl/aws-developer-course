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

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
# Section 12: AWS Integration and Messaging: SQS, SNS, Kinesis
-if want to deploy multiple apps need to config integration 
-SQS = oldest AWS service
-SNS, Kinesis for RT streaming of data

### Intro to Messaging
-middleware to orchestrate stuff between services
-deploying multiple apps means they need to communicate with one another 
-2 patterns of communication: 
1. **Sync** = app to app 
1. **async/event-based** = via middleware or queue that connects apps 

e.g. encoding service overwhelmed b/c of traffic it's better to **decouple** app and have decoupling layer scale for me 

can decouple using: 
* **SQS** --> queue model 
* **SNS** --> pub/sub model 
* **Kinesis** --> RT streaming model (for big data)

Paradigm: Our AWS services can scale independently via SQS, SNS, Kinesis from our app!

### AWS SQS 
**producers** send messages to an **SQS queue**, can have as many producers as want 
--> on other end = **consumer**, can have as many as want (doesn't have to be same number as producers)

SQS = oldest offering of AWS 

**STANDARD QUEUE**
-fully managed 
-scales from 1 message to 10,000 messages/s --> auto 
-message is data (like JSON)
-default retention = 4 days, set up max= 14 days 
-no limit of how many messages can be in queue
-low latency (<10ms)
-**horizontal scaling** for # of consumers
-**can have duplicate messages or out of order messages, rare tho (b/c super high throughput)**
-**max of 256KB/message**

**DELAY QUEUE**
-consumers don't see messages right away, **up to 15 min**
-default = 0s (producer sends message to queue message available right away)
-**can set new default at queue level** 
-can override default with **DelaySeconds parameter**

what do messages look like? 
**body** max of 256KB **has to be a string** 
* can add **message attributes** aka metadata --> OPTIONAL --> if provided, need Name, Type, Value of attribute 
* can provide **delay delivery** --> OPTIONAL 

When message sent to SQS queue --> get back **message identifier** (an ID) AND an **MD5 hash of body**

Consuming messages (consumer side)
-messages not pushed directly to consumers, instead consumers **Poll(ask) for SQS messages (max 10 at a time) and processes message**
-consumers must process the message within the **visibility timeout** 
-can delete message using **message ID and receipt handle** --> messages always deleted by consumer 

What is consumer processing of messages? 
-one example is to insert message into shipping message if the consumer is using a buying service

**visibility timeout** --> for a defined period, message polled = invisible to other consumers --> **between 0s and 12hrs**, default **30s**
-if set **TOO HIGH (15 min)** and consumer fails to process message --> must wait 15 min before processing again, bad consumer experience 
-if set **TOO LOW (30s)** and consumer needs 30s+ to process message and delete, another consumer will get message and message will be processed more than once 
* how do we set it? know what going to do so think about happy medium
* change timeout with **ChangeMessageVisibility API**
* delete message and tell SQS message successfully processed with **DeleteMessage API**

**DEAD LETTER QUEUE (DLQ)** 
if no processing within visibility timeout--> message back to queue and other consumers can get it --> if message goes back to queue over and over again it will hit a threshold called **redrive policy** 
* after threshold exceeded (message is faulty) --> goes to a Dead Letter Queue
* **DLQ** must be created first and then designated as such
* in DLQ need to make sure messages are processed before they expire 

**long polling aka receive message wait time** = when consumer req messages from queue it can optionally wait for some if none are available
* GREAT b/c **decreases number of API call made to SQS and increase efficiency and latency of app**
* wait time = 1-20s, 20s=better
* long polling > short polling
* enabled at queue level as default OR at API level via **WaitTimeSeconds API**

### SQS Hands On 
SQS --> get started --> standard queue 
* unlimited throughput 
* different orders of messages etc. 

see SSE settings for encryption using KMS
--> create queue 
-get URL, ARN, Name, etc. etc. 

queue actions dropdown has many options

righ click--> send message--> can delay up to 15 mins --> send --> now UI has message! --> view/delete message, see receive count increment 

### DLQ Hands On for redrive policy 
-might want to keep messages longer to make sure they're processed 
-need to set a redrive policy on 1st queue to talk and send messages to the DLQ we just created 

configure queue action --> redrive policy set
-see the redrive policy tab for policy 

### SQS CLI Practice
CLI can take many commands relating to SQS, many APIs that can be called 
-**aws sqs list-queues** --> will return nothing if CLI not configured with same region as SQS queue 
fix: **aws sqs list-queues --region [regionName]** 

--region allows for API calls against another region 

```
aws sqs [command] help
```
for any docs on command

to get messages: receive-message command for receive message API 
a lot of args BUT can see message in CLI! 
-message is in JSON 

-can see that console matches up with what is happening, fully integrated 
-to delete message need to provide receipt handle value we get from message JSON as an arg of the delete-message command

### FIFO Queues
-new offering, not in all regions 
-**name must be with .fifo extension** 
-**lower-throughput** but increased liklihood of ordered messages! therefore customer gets all messages in order
* throughput **either 3,000/s with batching or 300/s**
-messages sent 1x only 
-no per message delay (only per-queue delay)

Common exam Qs
**deduplication** = not sending message twice 
* each message to FIFO has **MessageDeduplicationId** field (attribute)
* if message sent twice then deduplication WILL happen on it --> interval = 5 min, if see the duplicate in this time then FIFO will delete one of the messages
* **content-based duplication** = MessageDeduplicationId is generated as the **SHA-256** (hashing algorithm) of the body (not attributes)

**sequencing** = to make sure perfect FIFO maintained need to have a **MessageGroupId**
* messages with diff group ID might be received out of order 

Hands On 
send message to FIFO queue

### SQS Advanced
**extended client** = JAVA library for sending larger messages (>256KB)
* leverages S3 on top of SQS queue --> send large message to bucket 
* producer will send a small metadata message to queue instead
* using library, consumer receives metadata and then consumer gets message from S3

SECURITY 
-encryption in flight using HTTPS endpoint 
-can enable SSE using KMS
* can set own **customer master key (CMK)** to use 
* can set data key reuse period (1min-24hr, less reuse means KMS API used a lot, pay more VS. more reuse, opposite, slightly less security) --> default = 5min 
* **SSE only encrypts the body, not metadata** 
-IAM policy must allow usage of SQS
**IAM gives perms for all API calls in account** 
-**SQS queue access policy** = finer grain control, can use IP address to decide who can access
-**no VPC endpoint** therefore must have wifi for accessing SQS

APIs to know for exam
* CreateQueue
* DeleteQueue
* PurgeQueue (to delete all messages in queue, takes up to a min)
* SendMessage
* ReceiveMessage
* DeleteMessage
* ChangeMessageVisibility (change timeout)

* Batch processing options for: SendMessage, DeleteMessage, ChangeMessageVisibility --> cheaper 

### SNS 
-if want to send message to many consumers
-can do direct integration but it's painful 
-alternative = **pub/sub** 
* subscribers get data from SNS topic (one buying service sends one message 1x to SNS topic and this topic sends it to all subscribers)

**event-producer** = what sends 1 message to SNS topic 
**event-receivers** aka subscriptions = what gets the event, as many of these as we want 
* each subscriber will be able to get all messages (filter messages too)
* up to 10,000,000 subscriptions/topic 
* 100,000 topics limit 
**subscribers** = many things, an SQS, an HTTP/HTTPS endpoint (with # delivery retires specified), Lambda, emails, SMS messages, mobile notifcations

Why use this? 
-clean architecture that intregrates with so many services 
* CloudWatch (alarms)
* ASG notifications 
* S3 (bucket events)
* CloudFormation (upon state changes)
* etc...

How to publish to SNS? 
**topic publish** (within AWS server using SDK) 
* create topic 
* create at least 1 subscription 
* publish to the topic 
**direct publish** (for mobile apps SDK)
* create platform app 
* create platform endpoint
* publish to endpoint 
* works with all notif deliveries for mobile

**Fan out** = common pattern with SNS and SQS
-want to push data once in SNS to topic and have it fan out to SQS queues
* fully decoupled
* no data loss 
* ability to add receivers later 
* SQS allows for delayed processing 
* SNS allows for retries of work 
* many workers on one queue or one worker on the other queue 
--> very popular exam question 

### SNS Hands On

### Kinesis Overview 
**kinesis** = management alternative to Apache Kafka --> big data streaming service that allows for collection of: 
* app logs 
* app metrics
* IoT 
* clickstreams 
--> anything RT big data 
-compatible with many streaming processing frameworks (Spark, NiFi, etc...) --> ways to preform computations in RT on data that arrives via stream 
-**kinesis data auto replicated to 3 AZs** 

**kinesis streams** = low latency streaming ingest at scale 
**kinesis analytics** = to preform RT analytics on streams using SQL 
**kinesis firehose** = to load streams into S3, Redshift, ElasticSearch from AWS 

streams = main focus on exam 
kinesis = made up of streams 
-get streams from data producers (click streams, IoT devices, Metrics and logs) 
-kinesis processes the big data in RT with analytics --> computations 
-when done, computations moved via Kinesis firehose to a storage location (S3 bucket, Redshift, etc.)
--> overall allows for quick onboarding of data en masse, analyze, and put into storage

**streams** 
* divided into ordered **shards aka partitions** (one little queue)
* to scale up stream add shards 
* **data retention is default 1 day (up to 7)** for shards
* **data can be reprocessed/replayed (vs. SQS once data consumed it's gone)**
* **multiple apps can consume the same stream** 
* RT processing with scale of throughput b/c can add shards! 
* once data inserted in Kinesis can't be deleted (aka **immutability**)

Shards
-one stream = many diff shards 
-1 **shard** = 1MB/s or 1000 messages/s at write and 2MB for read
-**provisioned**, so will pay for how many shards I have provisioned 
-can **batch** messages and calls, therefore can efficiently push into kinesis this way 
-**reshard/merge** = number of shards changing (increase = reshard, decrease = merge)
-**records are ordered/shard**

Put Records
**put records** = producer-side via Kinesis API 
-way to send data to kinesis
-need to send data via **PutRecord API**  in **partition key** that gets hashed --> used to determine shard Id (way to route data to specific shard)
* **same key always goes to the same partition** to help ordering for specific key 
* data only goes to one shard at a time --> when data at shard it gets a **sequence number** (number always increasing)
* **choose partition key that is highly distributed to help prevent a hot partition (when all data goes to same shard and shard is overwhlemed)** (e.g. many users but we can give a user_id as a partition key because it's distributed)
* can use batching with PutRecords to reduce cost and increase throughput
* if get exception called **ProvisionedThroughputExceeded** --> means over the limits, use retires, exponential backoff 

**ProvisionedThroughputExceeded** 
-get this exception when sending more data than shard can handle (exceeding MS/s or TPS for any shard)
-need to make sure don't have a hot shard(where partition key is bad)
-SOLUTION: 
* retries with backoff
* increase number of shards
* ensure parition key good 

Consumers 
-can use a normal consumer (CLI, SDK, etc.)
-OR can use **Kinesis Client Library (KCL)** --> Java, Node, Python, Ruby, .NET 
* uses DynamoDB to checkpoint offsets, track other workers and shre the work amoung shards
 
--> way for consumer to consume from kinesis efficiently 

### Kinesis Hands On 
-always have to pay for it 

-create stream --> ARN, etc. 
-one shard open 
-can enable SSE 
-can retain data for 24hr (or more with more money)
-can have shard-level metrics 

-in CLI aws kinesis help to show commands 
```
aws kinesis list-streams
```
see the list of streams 
-can see shard in CLI 

-when put-record --> returns ShardIf and SequenceNumber
-to retreive records --> get-shard-iterator and then get-records
-use the iterator to get the records! 
-data is **base-64-encoded** so to deal with it can do a base 64 decode freeware or programmatically 

### Kinesis KCL 
-kinesis client library = Java library that helps read records from streams with distributed apps sharing the read workload (shared workload between diff instances of app) 
**Rule: each shard is to be read only by one KCL instance**
* therefore, 4 shards = max 4 KCL instances
* **progress of KCL app is checkpointed into DynamoDB table (need IAM access to write to)**
-KCL can run on: 
* EC2
* EB
* On-premise app 
**records read in order of a shard but across shards we don't know order** 

**shard-splitting** = adding more shards --> therefore can re-scale the KCL apps and scale up the number of KCLs to match the number of shards

### Kinesis Security, Analytics, Firehose
Security
-need IAM for access/auth 
-encyption in flight using HTTPS endpoints 
-can enable encryption at rest (i.e. SSE) using KMS
-can encrypt/decypt client-side 
-can enable VPC endpoints to access kinesis within VPC 

Analytics
-RT analytics using SQL
-auto scaling 
-managed (no servers to provision)
-only pay for what we consume/compute (unlike streams that are provisioned)
-can create streams out of the RT queries

Firehose
-fully-managed
-near RT (1 min latency)
-used for loading data into Redshit/S2/Splunk/ElasticSearch
-auto scaling 
-support for data formats (pay for conversion)
-pay for the amount of data through hose 

### SQS vs. SNS vs. Kinesis 
-common Q = which is best to use when? 

SQS
-consumers **pull data**
-data deleted right after consumption 
-can have many workers as we want 
-auto scale throughput
-no ordering gaurantee (unless FIFO)
-each message can be delayed

SNS
-pub/sub paradigm 
-**push data** to many subs, up to 10 million 
-data not perisisted
-up to 10,000 topics
-auto scale throughput 
-integrates well with SQS via fan out pattern 

Kinesis 
-consumers **pull data**
-as many consumers as want, but only 1/shard 
-can reprocess data if want 
-RT, big data analytics and ETL = keywords
-ordering at the shard level 
-data expires after X days 
-must provision throughput! 
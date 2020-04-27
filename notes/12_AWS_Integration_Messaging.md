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
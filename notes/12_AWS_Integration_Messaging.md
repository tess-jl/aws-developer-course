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

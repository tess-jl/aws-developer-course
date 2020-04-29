# Section 14: AWS Serverless: DynamoDB 
-want a nice serverless database --> Dynamo DB 
-auto scales for me
-integrated with lambda 

### DynamoDB Overview
- noSQL, serverless DB

Traditional AWS app
-leverages **RDBMS database** relational, SQL
-strong requirements about how data should be modeled --> therefore can do joins, aggregations, etc. --> therefore scaling = vertical (more powerful CPU, RAM, I/O)\

NoSQL databases = new database
-**NoSQL** != only non-SQL, means not only SQL databases
-**non-relational** (can't do joins), **distributed** (**horizontal scaling**)
-i.e. MongoDB, DynamoDB
-all data needed for query is present in one row
-don't perform aggs such as "SUM"--> just give data need efficiently --> have to do aggs client-side

**DynamoDB** 
* fully-managed
* high available
* replication across 3 AZs by default
* scales to massive workloads b/c distributed, horizontal scaling 
* millions of req/s, trillions of rows, 100s of TB of storage
* fast and consistent --> low latency 
* integrated with IAM
* we just provision tables, don't see servers
* can do **event-driven programming** with DynamoDB streams
* low cost and auto scaling 

Basics
* made of **tables** --> each with infinite number of rows aka items --> each item has **attributes** (like columns), can be nested, added over time (some can be null)
* each table has a **primary key** (decided at time of DB creation)
* max size of row = 400KB 
-**Datatypes supported**: 
* scalar types (string, num, binary, boolean, null)
* document types (List, Map) as attributes
* set types (String set, num set, binary set), allows us to do nested stuff 

**primary key** options: 
1. **partition key only (HASH)** --> MUST be unique for each item, must be "diverse" so data is distributed across many partitions like userId
1. **partition key and sort key** --> more DynamoDB-specific, **combo must be unique**
--> data grouped by partition key 
--> data sorted by **sort key (aka range key)**
--> e.g. user_id and game_id because user can have many games but can only play one at a time 

Exam will ask what is parition key optimal for distribution? --> think rotten banana lab

### DynamoDB Basics Hands On 
-gives estimated cost to run table/month 
-can encrypt data at rest 
-create table
-overview tab --> all the provisioned stuff, ARN 
-items tab --> can add data to our table --> when create items and leave some fields blank they will be null, fine for any field to be null
**only field that cannot be null (i.e. required for every row) are the partition key and the sort key** 
-sort key as a timestamp makes things unique! 

### DynamoDB WCU and RCU - Throughput
-each table needs correct read and write capacity units 
**Read Capacity Units (RCU)** = throughput for reads 
**Write Capacity Units (WCU)** = throughput for writes 
* have option to set up auto scaling to meet demand of throughput 
* throughput can be exceeded temporarily via **burst credit** 
* if burst credits empty then will get a **ProvisionedThroughputException** --> then advised to do exponential backoff retry 

WCUs
* one WCU = **one write/s for an item up to 1KB**
* if item > 1KB --> more WCU consumed 
* e.g. write 10 objects/s of 2KB each therefore 20WCU needed! 
* e.g. write 6 object/s of 4.5KB each --> therefore need 30WCU
* e.g. write 120 object/min of 2KB  --> need 4WCU

RCUs
can either be **strongly consistent read** or **eventually consistent read** 
* **eventually consistent read = default** = when read just after write --> possible will get res that's outdated becuase of replication time but if keep asking will get right results 
* **strongly consistent read** = if read just after a write will always ger right data BUT it impacts RCUs a lot 
--> default = eventually consistent BUT **GetItem, Query, and Scan have ConsistentRead parameter that can be set to true**

**1 RCU = 1 stronly consistent read/s OR two eventually consisten reads/s for an item up to 4KB**
e.g. 10 strongly consisten reads/s of 4KB each--> therefore 10RCU 
e.g. 16 eventually consisten reads/s of 12 KB each --> therefore 24 RCU 
e.g. 10 strongly consistent reads/s of 6KB each --> 20RCU b/c have to round up to needing 8KB size, only increments of 4KB 

**internal partitions** 
* data divided into partitions 
* partition key goes through hashing algorithm to know which parition the data goes to 

**WCU and RCU are spread evenly throughout partitions**

**ProvisionedThroughputException**
-exceeded RCU or WCU
-why? 
* **hot keys** = one parition key being read too many times
* **hot partitions** 
* **very large items** (since RCU and WCU depend on size of items)
--> solution 
* exponential backoff, already in the SDK
* distribute partition keys as much as possible
* if RCU issue can use **DynamoDB Accelerator (DAX)**

### WCU and RCU Hands On 


REVIEW
differences between MongoDB and DynamoDB

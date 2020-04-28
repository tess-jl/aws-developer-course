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
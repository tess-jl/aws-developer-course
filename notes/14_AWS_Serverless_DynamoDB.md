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
-assign these units to table --> use the capacity calculator which helps determine capacities and estimated cost for workload 

**strongly consisten = 2x read capacity as eventually consistent** 
-if enable auto scaling then can't provision the units, it just auto scales it for us 

### DynamoDB Basic APIs 
Writing Data 
**PutItem API** = to write data to DynamoDB (create data or full replace) 
* consumes WCU
**UpdateItem API** = update data in DynamoDN (partial update of **attributes**)
* can also increase **Atomic Counters** 
**Conditional Writes API** = for conditional writes or updates, if condition not met then reject
* helps with concurrent access to items
* no performance impact 

Deleting Data
**DeleteItem API** = for deleting individual row
* can also perform conditional delete
**DeleteTable API** = deletes whole table and all its items 
* way faster than using DeleteItem on all items 

Batching Writes 
-efficient --> saves on latency (calls done in parallel), reduced calls done against DynamoDB
**BatchWriteItem API** = up to 25 items/call 
* either for PutItem or DeleteItem
* limit of 16MB of data written
* max 400KB data/item 
* possible for part of batch to fail --> **up to me to retry the failed items** again via exponential backoff

Reading Data 
**GetItem API** = to read based on primary key 
* primary key = HASH (key alone) or HASH-RANGE (key and sort key)
* eventually consistent by default 
* **if want only certain attributes (i.e. not the full item) in the res can use ProjectionExpression** --> saves in network bandwidth 
**BatchGetItem API** 
* up to 100 items
* max 16MB data 
* saves on latency b/c reading done in parallel

Querying data 
**Query API** returns items based on: 
* **PartitionKey** value (**must be a = operator**)
* **SortKey** value (with optional specification with **=, <, >, <=, >=, Between, Begin**) 
* **FilterExpression** for further filtering on **client-side**
--> returns: 
* up to 1MB data 
* or number of items specified in **Limit API**
* can do pagination on result s

-can query a table, local secondary index, or global secondary index 

Inefficient way of querying= **Scan API**
-every time do scan it scans the ENTIRE table, inefficient (consumes A LOT of RCU!)
* returns up to 1MB of data, can use pagination to keep reading
* can use a limit or reduce the size of the result and pause to keep from consuming so many RCUs
* faster = **parallel scans** --> multiple instances scan multiple partitions at same time, increases RCU and throughput --> faster but expensive --> limit or reduce the size of the result and pause to keep from consuming so many RCUs
* use a **ProjectionExpression and FilterExpression APIs** alternatively 

### DynamoDB Basic APIs Hands On
-shows how API calls are built into the UI
-clicking on id --> does a GetItem 
-in UI it preforms a scan under the hood
-can change scan to query --> nice GUI for a query
-Through console we can see whole range of APIs to use

### DynamoDB Indexes
**LSI (Local Secondary Index)** = alternate range key for table, **local to the hash key** 
* **must be defined at table creation**
* max 5/table
* sort key must be 1 scalar attribute 
* attribute chosen for LSI must be **string, number, binary**
-**no possibility to add/modify LSI**

**GSI (Global Secondary Index)** = new partition key + optional sort key
-like a whole new table that is updated based on what the base table has
-to speed up queries on non-key attributes 
-define a whole new index --> a new "table" the can project attributes on it 
* partition key and sort key from original table always projected (KEYS_ONLY)
* can also add extra attributes to project (INCLDES
* can use all attributes from main table (ALL
-must define RCU/WCU for the index 
-**possibility to add/modify GSI**

Indexes can cause Throttling 
**if writes throttled on GSI then base table will be throttled** --> true if WCU on base table still good 
* therefore, choose GSI parition key carefully
* assign WCU capacity carefully 

**LSI uses same WCU and RCU from base table, therefore no special throttling considerations**

Hands On 
-shows how Query in console can be done by Index using the LSI created when table was created
-**power of indexes = able to query on different things!**

### DynamoDB Optimistic Concurrency 
-often on exam 
-feature for conditional updates or deletes --> means that can be sure when we do an update or delete on an item it hasn't already been updated/deleted before altering it 
-therefore, DynamoDB = **optimistic locking /concurrency database** 

### DynamoDB DAX
**DAX (DynamoDB Accelerator)** = seamless cache for DynamoDB, no application re-write
* apps work the same when we enable DAX except NOW **all writes go through DAX to DynamoDB cluster**
* allows for **ms latency for cached reads and queries**--> solves the hot key problem (too many reads)
* default = 5min TTL (how long items live in cache)
* max 10 nodes/cache cluster
* multi AZ (3 nodes min recommended for production)
* secure (encryption at rest with KMS, VPC, IAM, CloudTrail...)
--> easy, simple way to improve app performance at minimal cost 

Hands On 
DynamoDB console --> create a DAX, select cluster size, etc. etc. 
-DAX not in free tier

### DynamoDB Streams
-changes in DynamoDb (create, update, delete) can end up in DynamoDB Stream --> changelog of the changes to the table --> stream read by Lambda
* with lambda can do whatever we want in RT in response --> welcome email, analytics, insert into ElasticSearch, etc. 
* could implement cross region replication using streams 
* only 24hr of data retention of stream 

Hands On 
-create a lambda function to react to the stream 
-see DynamoDB trigger section 
-go to IAM service to make sure this lambda function gets a new role --> add policies accordingly 
-designer GUI should show DynamoDB for lambda function --> see last processing result = OK therefore trigger worked 
-can also see that Cloudwatch logs shows this lambda function logs 
* can open up the logs and look at the details of the logs!!

### DynamoDB TTL 
**TTL (Time to Live)** = auto delete an item after an expiration date/time 
* define a column and based on columns the items that have an expiration date will expire and get deleted
* **free, deletions do not use WCU/RCU**
* background task done by Dynamo service itself 
* **helps reduce storage and manage the table size** 
* **TTL enabled/row** --> define a TTL column and then DynamoDB deletes them within a given time 
* typically deletes within 48hrs of expiration 
* deleted items also deleted in GSI, LSI 
* streams used for recovery of these deleted items

Hands On 
time to epoch --> epoch converter freeware --> get timestamp 
-create a table with some items set to expire at timestamps 
**Time to Live Attribute** = config where we list the column to look for TTL 
-can preview the TTL --> enable TTL

### DynamoDB CLI 
**--projection-expression** = list of attributes to retrieve from table, maybe just a subset of attributes 
**--filter-expression** = filtering of table 

general CLI paginaiton options for DynamoDB / S3
**Optimization**
* **--page-size** = for getting full dataset but each API call will req less data (helps to avoid timeouts)
**Pagination** 
* **--max-items** = for max number of items in res --> get a token in res called **NextToken**
* **--starting-token** parameter in CLU = to specify the last received NextToken to go to next page

Hands On 
```
aws dynamodb scan --table-name [name] --projection-expression "[column], [column]" --region [region]
```
therefore only getting a couple of the columns from this table! 
```
aws dynamodb scan --table-name UserPosts --filter-expression "user_id = :u" --expression-attribute-values '{ ":u": {"S":"124usersoi3"}}' --region us-east-1
```
should filter fust for one result 

etc. see the code/dynamodb dir for more

### DynamoDb Transactions
-new Nov 2018 
-similar to transaction in PostgreSQL 
**transaction** = ability to C, U, D multiple rows in different tables at the same time 
* **all or nothing** operation 

write modes = standard, transactional
read modes = eventual consistency, strong consistency, transactional 

transactional mode **consumes 2x WCU/RCU**

What is it and when should we use it? = for exam 
e.g. account balance and bank transaction table = use case --> write to both tables or none 

### DynamoDB Security and other features
Security 
* VPC endpoints available (i.e. dynamoDB without internet)
* full-acess controlled by IAM 
* encryption at rest via KMS
* encryption in transit via SSL / TLS 

Backup and restore features 
* point in time restore (like RDS), no performance impact 

Global tables (not just one region)
* multi-region, fully-replicated, high performance (relies on streams)

can use amazon DMS to migrate data to dynamoDB (from Mongo, Oracle, mySQL, S3, etc.)

can launch a local dynamoDB on laptop for dev



REVIEW
differences between MongoDB and DynamoDB
* attribute chosen must be **string, number, binary** for LSI --> attribute of what? 




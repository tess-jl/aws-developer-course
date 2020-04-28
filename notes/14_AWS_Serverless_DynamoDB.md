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


# Section 5: AWS Fundamentals Route 53 + RDS + ElastiCache + VPC 
-how to structure app in a 3 tier web app formation 

### AWS Route 53 Overview 
-**Route 53** = managed DNA (Domain Name System)
-collection of rules, records --> help clients understand how to reach a server through URLs 
-most common **records** are: 
* **A** = URL to IPv4
* **AAAA** = URL to IPv6 
* **CNAME** = URL to URL 
* **Alias** = URL to AWS resource 

-Route 53 --> helps browser figure out what IP to use to access the server --> does the mapping and communicates the back to browser (i.e. A record browser sends URL to Route 53 and then Route 53 sends back IP so that browser can use the IP in the HTTP req)
-Route 53 !== needed for every request --> IPs are cached so now browser knows how to access the app server 

-Route 53 can use public domain names (own or buy) OR private domain names (resolved by instanes in your VPCs)
-Route 53 has Load Balancing --> through DNS aka **client-side load balancing**, health checks, routing policy
-**use Alias records over CNAME records for AWS resources (for performance reasons)**

### Route 53 Hands On 
-in browser where we have the long URL from our LB and EC2--> don't want this crazy URL, want a nice one tht makes sense 
- need to register a domain name (cost money!! see video)
- hosted zone was created for us b/c domain name created via AWS
- NOW create a record --> go to LB --> LB's DNS name --> change this to the domain name
-DNS name in EC2 console is an A record and BEST practice is to set up an Alias record over CNAME!
-Choose A record --> say Alias --> redirects to LB as an Alias --> whatever.com --> value is an alias for the LB 

### RDS Overview
-db service 
-relational db service --> db is managed by AWS --> allows us to create dbs with SQL --> can create Portgres, Oracle, MySQL, MariaDB, Microsoft SQL, Aurora (AWS proprietary db)

Why use RDS vs deploying DB on an EC2? 
-managed service
* OS patching
* continuous backups (point in time restore)
* monitoring dashboards
* read replicas 
* multi AZ setup for disaster recovery (DR)
* maintenance windows for upgrades 
* scaling vertically or horizontally 
-BUT cannot SSH into instances, don't interact with them directly 

Read Replicas for read scalability 
- DB maybe needs to read to app a lot! --> creates up to 5 read replicas
-can be within AZ, cross AZ, or cross region 
-replication is async, reads eventually consistent but not right away 
- replicas could be promoted to their own DB if you want 
- **apps must update the connection string to leverage read replicas** --> only one instance takes the writes 

Multi AZ (for DR)
-sync replication --> one DNS Name (represented by RDS DB) in one AZ will hve an auto app failover to a standby in another AZ
-increases availability in case loss of AZ, loss of network, loss of instance or storage issue
-failover from master to standby --> auto, app still reads and writes to one DB 
-NOT used for scaling 

RDS Backups
-auto for RDS
-every night, full snapshot of DB
-capture transaction logs in RT 
-ability to restore to any point in time 
-7 day retention (up to 35 day if want)
-manual DB snapshot --> can retain backup for as long as want 

RDS Encryption
-encryption at rest via KMS - AES-256 encryption 
-SSL certificates to encrypt data to RDS in flight 
-**enforcing SSL** 
* PostgreSQL --> rds.force_ssl=1 in the AWS RDS Console (Parameter Groups)
* MySQL --> GRANT USAGE ON *.* TO 'mysqluser'@%' REQUIRE SSL;
- to connect using SSL: provide SSL trust cert (downloaded from AWS) and provide SSL options when connecting to DB 

-connecting via SSL !== enforcing SSL necessarily --> there is a difference

RDS Security 
- RDS dbs usually deployed within private subnet
- security works by leveraging security groups --> controls who (which IP, with other security groups, etc.) can communicate with RDS 
- IAM policies help control who can manage AWS RDS
- traditional username/password can be used to login to db --> ALSO for login, IAM user can be used for MySQL or Aurora 

RDS va Aurora 
-not open source 
-compatible with postgres and mySQL --> drivers will work as if Aurora was Postgres or MySQL
-Aurora = AWS cloud optimized, claims 5x performance boost compared to MySQL, 3x compared to Postgres on RDS
-Aurora storgae auto grows increments of 10GB, up to 64 TB 
-can have 15 replicas instead of just 5 (and replication is faster)
-failover is instantaneous for Aurora (high availability native, multi AZ)
-cost more than RDS, but more efficient so should even out theoretically 

### RDS Hands On 
standard create vs easy create 
--> standard --> 6 different engine types --> MySQL for this hands on --> templates etc.
-**SQL electron** = DB client, GUI to connect to DB --> .dmg package to install on Mac --> can connect to our RDS via the SQL Electron GUI with name/password

### ElastiCache Overview 
-ElastiCache is used to manage Redis or Memcached technologies
-caches = in-memory dbs with really high performance, low latency --> help reduce load off of dbs for read-intensive workloads
-helps make app stateless --> stores state in cache 
- **write scaling** via sharding 
- **read scaling** via read replicas 
- **multi AZ** with failover 
- AWS takes care of OS maintenance, patching, optimization, setip, config, failure recovery, backups --> a lot like RDS
-pretty much an RDS for caches (see bold above)

Solution architecture - DB cache fits in how? 
-apps query Elasticache first (if there cache hit) --> if query not availabe (a cache miss) then it gets it from RDS (reads from DB) and stores it in ElastiCache (writes to cache)
-helps relieve load in RDS
-cache must have invalidation strategy to make sure most current data used (app considers this)

Another architecture consideration: User Session Store 
-app is stateless --> user logs in --> app writes session data into ElastiCache --> user hits another EC2 instance --> app needs to know that user is logged in so it retrieves the user session off of ElastiCache to check that user is in fact logged in 
-so app can be stateless because the logged in state is managed by ElastiCache

Redis Overview
-in-memory key-value store 
-super low latency 
-cache can surive reboots (persistence)
-great to host user sessions, leaderboards, distributed state, relieve pressure on dbs, pb/sub for messaging
-multi AZ with auto failover for DR if don't want to lose cache data 
-support for read replicas 

Memcached Overview
-in-memory object store 
-cache !== survie reboots
-great for quick retrieval of objects from memory, cache objects
-Redis is really preferable but AWS still supports Memcached

### ElastiCache Hands On
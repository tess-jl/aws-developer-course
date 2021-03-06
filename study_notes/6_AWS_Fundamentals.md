# Section 6: AWS Fundamentals Amazon S3
- S3 powers the biggest websites in the world
-advertized as an infinitely-scaling storage
-used as an integration as well

### S3 Buckets and Objects 
- **Buckets** = giant directories where we store objects (files)
* need to have globally-unique name despite S3 being a regional service 
* naming convention: no uppercase, no underscore, 3-63 char, not an IP, must start with lowercase letter or number 

**Objects** = files 
* identified by a **key** = FULL, absolute path
* no actual concept of directories that objects live in but UI will make it seem that way (it's actually a very very long key name, lots of /)
* **Object Values** = the content of the file, the body --> max size 5TB, **if uploading more than 5GB need to use "multi-part upload"** 
* **Metadata** = list of text key/value pairs - system or user metadata
* **Tags** = key/value pair, unicode, up to 10, useful for security or lifecyle
* **Version ID** only if versioning is enabled

Create a bucket--> add coffee file to it
-if right click open it will evaluate permission as an AWS user but by clicking the Link (public link) --> 2 ways to open a file
-can create a folder
-have two keys --> coffee.jpg and Image folder with coffee.jpg

### S3 Versioning 
-Versioning = bucket-level setting --> means that if we overwrite a file we will update its version 
-best practice to version buckets(protect against deletes, roll back is possible)
-any file that was not versioned prior to enabling versioning will have the version null

Hands on
-if delete file looks like it's gone but if you show versions see that there is a delete marker on it (not truly deleted)

### S3 Encryption 
-Exam loves this section 
4 Methods for encypting objects in S3: 
1. **SSE-S3** --> encrypts S3 objects using keys handled and managed by AWS
1. **SSE-KMS** --> same but AWS uses Key Management Service to manage the encryption keys 
1. **SSE-C** --> you manage your own keys via AWS
1. **Client-side** --> manage everything client-side

**SSE (server-side encryption)**
**SSE-S3** --> object encrypted server side, we never see the keys, handled by AES-256
* MUST set header **"x-amz-server-side-encryption":"AES256"** on the HTTP/HTTPS req to S3, creates S3 managed data key --> encrypted--> put into the bucket

**SSE-KMS** --> also encryption server-side 
* keys handled/managed by KMS therefore more control over key audit trail
* MUST use header **"x-amz-server-side-encryption":"aws:kms"** --> creates a Customer Master Key (CMK) for the encryption before putting in bucket

**SSE-C** --> server-side using data keys fully managed by customer outside of AWS
* Amazon does not store encryption key 
* HTTPS must be used!
* encryption key needs to be in headers of every single HTTPS req made
* Amazon does the encrytion but throws away key from the req right away 

**Client-side** --> client-side obv 
* client library such as amazon S3 encryption client is used 
* clients also need to decrypt the data when retrieving it from S3 
* customer fully manages the keys and encryption cycle 

Encryption in transit (SSL)
-S3 exposes: 
* HTTP endpoints for non-encrypted traffic
* HTTPS endpoints when encryption is in flight 
-HTTPS is recommended (happening via SSL)
-HTTPS mandatory for SSE-C

-Encryption in flight = **SSL (Secure Sockets Layer) / TLS (Transport Layer Security)**

### S3 Security and Bucket Policies
**User-based** --> IAM policies - which API calls should be allowed for a specific user from IAM console
**Resource-based** --> more popular, done via: * **Bucket Policies** = bucket-wide rules from the S3 console, allows cross account 
* **Object Access Control List (ACL)** --> finer grain, only certain people need access to certain objects in bucket
* **Bucket Access Control List (ACL)** --> less common

S3 Bucket Policies 
-JSON-based policies need:
1. Resources: buckets and objects 
1. Actions: set of API to allow or deny 
1. Effect: allow/deny
1. Principal: account or user to apply policy to
-can use to: 
1. Grant public access to bucket 
1. Force objects to be encrypted at upload
1. Grant access to another account (cross account)

S3 Security cont. 
* **Networking** --> supports VPC endpoints (for instances in VPC without www internet)
* Logging and Auditing --> S3 access logs can be stored in another S3 bucket (NOT in same bucket due to recursion issues) --> API calls can be logged in AWS CloudTrail
* User Security --> enable MFA can be used in versioned buckets to delete objects 
* **Signed URLs** --> URLs that are valid only for a limited time (e.g. premium video service)

Hands On
-want to make it so bucket rejects a file that is not encrypted 
-Bucket Policy Editor --> need to type some JSON --> can use policy generator to write JSON from a UI 
-bucket policy = good way to ensure that encryption is happening 

### S3 Websites
-can set up static websites that are accessible to anyone on the web 
-URL will be either: 
1. [bucket-name].s3-website-[AWS-region].amazonaws.com
1. [bucket-name].s3-website.[AWS-region].amazonaws.com
-depends on region 
-if get a **403 forbidden error --> make sure bucket policy allows public reads**

Hands On: 
-creating a very simple S3 static website 
-go to permissions and add static website hosting for index.html --> get 403!
-need to add bucket policy!!!

### S3 CORS (Cross Origin Resource Sharing)
**CORS** = allows me to limit the nymber of websites that can request my files in S3 (and limit my costs)
- **CORS MUST be enabled if data is requested from another S3 bucket!** popular question
- if client (browser) is on site and tries to access file from another bucket and it's not working it's likely that CORS is not enabled properly or not at all

### S3 Consistency Model 
2 types of consistency: 
1. **Read after write for PUTS of new objects**--> as soon as the object is written we can retrieve it (e.g. PUT 200 and then GET 200) --> true EXCEPT for when we do GET before to see if object existed (GET 404 --> PUT 200 --> GET 404 because eventually consistent (will need to wait for cache))
1. **Eventual Consistency for DELETES and PUTS of existing objects** --> if we read an object after updating, might get older version (PUT 200, PUT 200, GET 200 could be older version) --> also if we delete an object might still be able to GET it for a short time afterward (DELETE 200, GET 200 ... GET 404)

### S3 Performance 
- historically, > 100 transaction/second S3 performance degredation 
- behind the scenes each object --> S3 partition --> want highest partition distribution
- used to be recommended to have random chars in front of key name (**prefix key**) to optimize performance --> forces AWS to partition data correctly 
- recommended to NEVER use dates as prefix keys 

NOW can scale up to 3500 RPS / PUT and 5500 RPS / GET for EACH prefix! therefore don't need to randomize object prefixes anymore to get faster performance 

cont. 
-faster upload of large objects (>= 100MB) --> use **multipart upload**
* parallelizes PUTs for greater throughput 
* maximize network bandwidth and efficiency 
* decrease time to retry in case of fail
* MUST use if size > 5MB
-can use **CloudFront** to cache S3 objects around the world therefore improves reads
-can use **S3 Transfer Acceleration** (uses edge location) --> just need to change the endpoint you write to, not the code
-**if have SSE-KMS encryption --> may be throttled, NOT by S3, but KMS service is throttling you** --> cannot keep up with the rate at which need to encrypt or decrypt data --> therefore need to change the KMS limits!

### Glacier SQL Select 
-theory 
-Glacier = another S3 storage tier that is meant for longterm archival of files 
-retrieve data from S3 in glacier --> may want only a subset of it, NOT all of it or else network cost is high 
-use S3 select aka Glacier select --> use SQL SELECT queries to let S3 or Glacier know exactly which attributes / filters you want
-80% of cost saved, performance increase of 400%
-select happens within S3 or Glacier
-works with CSV, JSON, or Parquet files 
-files can be compressed with GZIP or BZIP2
-No subqueries or Joins supported
-Have to do this within code, not code

REVIEW
SSL / TLC
How Computers Work by Ron White 
S3 --> **Simple Storage Service** --> S3 objects can be accessed from any EC2 instance (S3 is just storage, no processing power)
Edge locations (kinda between regions)
Bucket Policy vs IAM policy 

recommend multi-part for over 100MB 
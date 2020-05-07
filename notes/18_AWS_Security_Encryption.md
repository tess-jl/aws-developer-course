# Section 18: AWS Security and Encryption: KMS, Encryption SDK, SSM Parameter Store, IAM and STS
-bring together all aspects of security that we've already covered
-focus on KMS, etc. 
-practice with Lambda

### Encryption 101 
**encryption in flight (SSL)** = for sending sensitive info --> data encrypted before sending it and decrypted by server once received 
* **SSL certificates** help with encryption sent via **HTTPS** over the network
* protects against the MITM (man in the middle attack)

**SSE at rest** = when data encrypted after being received by the server 
* don't want someone who hijacks the sever to access the data 
* data decrypted before being sent back to client 
* all data stored in an encrypted form thanks to a key (usually called a **data key**)
--> encryption and decryption keys must be managed somewhere, usually a **key management system (KMS)** that the server must have access to 

**Client side encryption** = when data is encrypted by client and never decrypted by the server --> data stored on server but the server doesn't know what the data means 
* data decrypted by reciving client
* best practice = server should not be able to decrypt the data
* could leverage **envelope encryption** 

### KMS Overview 
**key management system (KMS)** = a store, an easy way to control access to data (AWS manages keys)
* behind most encryption within AWS 
* fully integrated with IAM for authorization 
* can use CLI / SDK to perform encrytion with KMS

integrated with: 
* EBS for encryting volumes
* S3 SSE of objects 
* Redshift encrypting data 
* RDS encrypting data 
* SSM Parameter store 
etc... 

Anytime need to share sensitive info (i.e. DB passwords, creds, private key of SSL cert) --> use KMS 
* **Customer Master Key (CMK)** = used to encrypt data, managed by KMS --> can NEVER be retrieved by the user --> can be rotated for extra security 

NEVER store secrets in plain text, code
* can send secrets to KMS to encrypt them and then encrypted secrets could be stored in code / env vars 
* only people who can decrypt secrets are those with IAM perms to do so via KMS 
* **KMS can only encrypt max 4KB data/call**

data > 4KB --> MUST use envelope encryption 

give someone access to KMS:
* **key policy allows the user**
* **IAM policy allows the API calls** 

KMS gives us ability to manage keys and policies (even though we never see keys themself), can: 
* Create keys
* Set Rotation policies 
* Disable keys 
* Enable keys 
* Audit key usage via CloudTrail
--> 3 types of CMK: 
1. **AWS Managed Service Default CMK** = free 
1. **User Keys created in KMS** = $1/month 
1. **User Keys imported** = must be 256-but mymmetric key, $1/month 
AND pay for APU calls to KMS ($0.03/10,000 calls)

How does KMS work? 
-many APIs--> Encrypt API and Decrypt API: 
* if we have a password --> use Encrypt API to send secret to KMS and we use a CMK in KMS to encrypt (KMS checks if client has perm via IAM, performs encryption) --> KMS performs encrypted secret!
* if we want to decrypt the password --> make API call to Decrypt API --> goes to KMS and uses same CMK for encryptions (checks IAM perms, does decryption) --> KMS sends back decrypted password 

### KMS and Lambda Practice 
KMS console --> use KMS to encrypt and decrypt secrets for a lambda function 
**AWS managed keys** --> created and managed by AWS whenever we enable encryption for a service (we don't have direct access to these keys!!)

**Customer managed keys** = where we will create a key via KMS --> etc etc --> review and edit key policy (JSON) that shows permissions, what IAM permissions are enabled to allow access to the key itself--> finish
* see ARN
* alias 
* status, etc. 

create lambda function --> before the handler use the env var and make a decrypt call for the variable so that it is secure --> API calls won't work until we add KMS perm for the IAM role, add inline policy for this resource (use ARN for our key)--> now when we test lambda function it works --> checkout cloudwatch logs for what happened! 

### Encryption SDK Overview
-need to know when we should be using it 
-if KMS hasa a limit of 4KB how does it do it? 
--> via envelope encryption, very cumbersome to implement 
* instead use **AWS Encryption SDK** that helps us use Envelope Encryption (VERY different from the **S3 encryption SDK**) --> can be used as a CLI tool! 

Exam--> know that anything > 4KB of data that needs to be encrypted to use Encryption SDK, Envelope Encryption, **GenerateDataKey API**

*How does Encryption SDK work?*
-client uses CLI or SDK, has a big file --> make GenerateDataKey API call to KMS --> KMS checks IAM perms good then 1) generates Data Key (DEK) and 2) encrypt data key with CMK --> **plain text data key and encrypted data key** sent back to client

plain text data key = for client-side encryption --> get an encrypted file via this data key --> NOW delete the plain text DEK --> file file has the encrypted file and the encrypted DEK 
--> because 2 levels of encryption = envelope encryption

*How does Decryption of envelope data work?*
client with encrypted data makes call to Decrypt API in KMS--> KMS will check IAM perm and KMS will decrypt using CMK and send back the decrypted, plain text data key to client 

client uses plain text DEK to decyrpt the big file client side 

### Encryption SDK Hands On 
https://aws-encryption-sdk-cli.readthedocs.io/en/latest/

install the AWS encryption SDK for CLI 
```
pip install aws-encryption-sdk-cli
```
use full ARN of our key to run commands 
-encrypt and decrypt commands --> see example for how used 

metadata = JSON doc. can be used for summary of what happened but not needed

### SSM Parameter Store Overview 
**AWS Systems Manager Parameter Store Service** = secure storage for config and secrets (e.g. DB passwords), way to centralize params for AWS account
* one of the most underutilized services in AWS
* OPTIONAL **Seamless Encryption** using KMS
* serverless, scalable, durable, easy SDK, free
* version tracking of configs/secrets 
* config management done via IAM (restrict who can use which passwords) and using path
* notifs with CloudWatch Events
* integration with CloudFormation

store can be used to do encrypted configuration using KMS behind the scenes --> therefore simplified way of using KMS 

*Store Hierarchy* 
the scaffolding pattern 
-can be organized however we want but should be a convention! 
**hierarchy convention used with the GetParameters API or GetParametersByPath API** with a lambda function, for example 
--> therefore can differentiate diff params for diff envs 

### SSM Parameter Store Hands On (CLI)
system manager --> parameter store 
param in demo = db URL --> type = String, value = some URL --> create param 

create another param --> DB password --> type = SecureString (therefore encrypted), use AWS KMS key or key made before --> value = some password --> create param

create another DB URL param and password but for prod env

--> now 4 params, we can access them using the CLI 
```
aws ssm get-parameters --names [paramNames]
```
can see how the password is encrypted --> can use the special ---with-decryption in the command and now the password in the res will be decrypted! will only work if you have access to KMS 

```
aws ssm get-parameters-by-path --names [pathName]
```
--> will query for all params under that path (why the tree is nice! can get all the secrets at once)

### SSM Parameter Store Hands On (Lambda)

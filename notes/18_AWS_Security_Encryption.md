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




# Section 7: AWS CLI, SDK, IAM Roles and Policies 
-So far we've manually interacted with services to see standard info (all but S3)
1. How do we develop against AWS without using the online console? reduce manual experience 
1. How do we interact with AWS proprietary services (S3, DynamoDB, etc)

Can be done in serveral ways: 
* AWS CLI on local computer 
* AWS CLI on EC2 machine 
* AWS SDK on local computer 
* AWS SDK on EC2 machin
* AWS Instance Metadata Service for EC2

### CLI Setup 
https://docs.aws.amazon.com/cli/latest/userguide/install-macos.html#install-bundle-macos

### CLI Config
-CLI will allow my laptop to access the AWS Network
-learn how to get access creds and protect them 

Use IAM service to get access cred to config CLI!
-"aws configure" command in CLI --> enter id and secret key --> creates 2 files --> one config and one credentials 

### CLI Practice with S3 
https://docs.aws.amazon.com/cli/latest/reference/s3/
```
aws s3 ls 
```
to list all s3 buckets 

```
aws s3 ls s3://[nameOfBucket]
```
to list contents of the bucket

```
aws s3 cp help
```
to read documentation


to copy something from s3 to computer or vice versa: 
```
aws s3 cp s3://[nameOfBucket]/[nameOfFile] [nameOfFile]
```

Most commands make sense intuitively 

### AWS CLI on EC2
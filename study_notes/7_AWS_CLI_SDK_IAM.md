# Section 7: AWS CLI, SDK, IAM Roles and Policies 
-So far we've manually interacted with services to see standard info (all but S3)
1. How do we develop against AWS without using the online console? reduce manual experience 
1. How do we interact with AWS proprietary services (S3, DynamoDB, etc)

Can be done in serveral ways: 
* AWS CLI on local computer 
* AWS CLI on EC2 machine 
* AWS SDK (Software Development Kit) on local computer 
* AWS SDK on EC2 machine
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
-common to run the CLI on EC2--> there's bad way and good way 
1. Bad way: if we run aws configure on EC2 --> BUT it's super insecure, NEVER EVER put personal credentials on an EC2 (we never own an EC2, don't want ANYTHING personal on any EC2)
1. **Good way: via IAM roles**

IAM roles can be attached to EC2 instances
-roles can come with a policy--> the policy can define exactly what the EC2 instance should be able to do (default is no rights)
-with instance having its own role it's way more secure 
-onlu 1 IAM role at a time (but role can have many policies)

Hands on
-go to EC2 console on AWS
-SSH into the EC2 instance 
-aws configure once in EC2 instance but DO NOT put any access key or secret! just enter default region name 
-if try to run any command it will try to point us to aws configure but really we should be using Instance IAM Roles! 

--> go to IAM --> Roles --> create role --> want AWS Service as entity since we want to attach it to an EC2 

-roles make it so that any service can have its own set of permissions

-create role (use built in policy for read-only amazon s3)
-go back to EC2 instance --> right click to role settings--> add new role 
-NOW when go back to CLI can do aws s3 ls and the command works! 
--> but if we try to create a random bucket in the CLI --> WILL NOT WORK b/c EC2 role is not permissioned enough to do a make bucket operation 
-can click on the role and add another policy --> such as S3 Full Access --> with this policy we can now make a bucket 

### IAM Roles and Policies Hands On
-can either attach a built-in policy (AWS-managed and auto updated) or can create our own with these parameters:
* Service
* Actions 
* Resources 
* Request conditions

-can also add an in-line policy (on top of what I already have for policies) --> unique to ONE role only, not as nice b/c better to manage policies globally

This s3 read-only policy: 
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:Get*",
                "s3:List*"
            ],
            "Resource": "*"
        }
    ]
}

^JSON like this says that for ANY AWS S3 resource (*) this policy on the role allows the resource to perform an API calls that start with "Get" or "List" --> see ListBucket API 

This s3 full access: 
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": "*"
        }
    ]
}


Building our own Custom Policy: many options for an S3 policy! many API calls --> can add a specific ARN for a policy just for a bucket 
* did this on the standard visual editor 
* can also use AWS policy generator GUI 

Advantage of custom policies: can see who is using it and the versions of the policy --> because managed by me I can make it more specific to my EC2 instance the security can be even better!

### AWS Policy Simulator 
-want to test the S3 read-only policy we have 
-Policy Simulator tool https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html
- simulation of what can be done --> kind of similar to postman but simulation of what is allowed
-easiest way to test policies!

### AWS CLI Dry Run 
-don't want to run the actual commands, just want to make sure we have the permissions
-some commands (not all) have a --dry-run option to simular API calls
e.g. 
```
aws ec2 run-instances help
```
shows us docs on this command
```
aws ec2 run-instances --dry-run --image-id ami-[id number] --instance-type t2.micro
```
--> will show that not auth to use this command!
so, add permission to make it so that we can run-instances by editing the custom policy we made
-make sure custom policy is attached to the role 
-should see (DryRunOperation) in CLI that indicates we could run that op but we won't due to the --dry-run 

### AWS CLI STS Decode
-we can get a long error message in the CLI --> how do we decode it? Using the **STS command line**
-need to run:
```
sts decode-authorization-message --encoded-message [value]
```
-get access denied --> because IAM role is not auth to preform operation --> go to custom policy and update it with STS action--> write decodeauth --> returns JSON that can be prettified into a nice error message! 

### EC2 Instance Metadata
-least known 
-allows EC2 instances to learn about themselves --> don't need to use an IAM role for that purpose
-URL is **http://169.254.169.254/latest/meta-data**
-this IP is an internal IP to AWS --> does not work from computer, just works from within EC2 
-**cannot get content of the IAM policy but can get role name**
-**metadata** = info on EC2
-**userdata** = launch script of the EC2

Hands on
```
curl http://169.254.169.254
```
version of API call we're using --> we just want latest meta data so: 
```
curl http://169.254.169.254/latest/meta-data/
```
gives us list of data--> / means there is more
```
curl [thingFromList]
```
returns the data 

ANY EC2 instance, even without IAM role can get this metadata this way 
```
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/[nameOfRole]
```
shows access key and secret--> SO behind the scenes--> EC2 role gets temporary creds via role attached

-useful for automation of EC2 instances

### AWS SDK Overview
-if want to preform actions on AWS directly from app's codebase (i.e. no CLI)?
-**Software Development Kit (SDK)** 
-official SDK:
* Java 
* .NET
* Node.js
* PHP 
* Python (renamed boto3/botocore)
* Go 
* Ruby
* C++
-most languages support AWS SDK

-SDK useful when start coding against AWS service like DynamoDB
-AWS CLI is a wrapper around Python SDK (boto3) (hence download with python in it)
-exam expects I know when to use SDK
-with the SDK --> if no region specified default is us-east-1

CDK Credentials Security 
-recommended use **default credential provider chain** --> works perfectly with aws credentials --> will auto look for credentials in this file on my machine
* can also use **Instance Profile Credentials** --> use IAM Role for EC2 machines etc.
* can also use .env vars, less-recommended!!

**Exponential Backoff** = a retiring strategy of any API that has failed because of too many calls (only applies to rate-limited APIs) --> strategy = wait 2x as long for each API call on retry, ensures not overloading API 
-retry mechanism is included in SDK API calls

### AWS CLI Profiles 
-manage multiple AWS accounts? 
-need to use a **profile** --> need to define a name for a new profile, configure it, can then use it from any CLI 
```
aws configure --profile [profileName]
```
then prompted with new form --> enter our special AWS credentials from our new account

```
cat credentials
```
shows all accounts credentials 
```
cat config
```
lists both accounts

to execute a command for that other account need to use:
```
[command] --profile [accountName]
```

REVIEW 
aws configure and credentials --> Q3 quiz
-can't attach EC2 IAM roles to on-premise servers
-policy simulator great for comparing policies

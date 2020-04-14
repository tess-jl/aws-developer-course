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
-common to run the CLI on EC2--> there's bad way and good way 
1. Bad way: if we run aws configure on EC2 --> BUT it's super insecure, NEVER EVER put personal credentials on an EC2 (we never own an EC2, don't want ANYTHING personal on any EC2)
1. Good way: via IAM roles 

IAM roles cna be attached to EC2 instances
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

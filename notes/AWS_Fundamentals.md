# Section 3: AWS Fundamentals: IAM + EC2 

AWS regions --> each region has availability zone (e.g. us-east-1 a), represents different parts of the world, isolated from disasters

-AWS consoles are region-scoped (excpt IAM and S3)

### IAM Service overview
-identity and access managemnet
1. users
1. groups
1. roles 
-root account should never be used (and shared)
-every service in AWS helped by IAM
-users must be created with proper permissions
-policies written in JSON 

-users get an account in IAM (not root), users can be grouped (usually be teams, etc.) into groups --> can apply permission to groups and users inherit 
-roles what we give to machines 
-policies = JSON docs that define what users, groups, roles can do

-IAM has global view --> permissions via Policies(JSON)
-multifactor auth can be set up 
-best to give users minimal amount of permissions needed to do job (least privilege principles)--> don't over power any person, app, or server 

-Big enterprises use IAM Federation (when a company uses AWS)--> one can log in with company credentials --> identity feds use the SAML standard(active directory)

1 IAM user / physical person 
1 IAM role / app
IAM credentials KEPT SECRET- NEVER WRITE IN CODE!
Never use root account except for initial setup--> never use ROOT IAM CREDENTIALS! 

### IAM hands on 
-set up two factor auth for root 
-create a user 
-create a group 
-manage/assign permissions
-create an alias
-sign in again with alias to use AWS

### EC2 Service
-one of the most popular services 
-EC2 consists of:
1. renting virtual machines (EC2)
1. Storing data on virtual drives (EBS)
1. Distributing load across machines (ELB)
1. Scaling the services using an auto-scaling group (ASG)
-fundamental to understanding how cloud works 

-launch EC2 instnce running Linux
-state / stop / terminate instance
-when start an instance have to have its operating system somewhere --> it's on a disc --> called storage = EBS volume 
-when launch an instance can have tags (key/value pairs which allow you to identify instance and classify it)
-security group = firewall around the instance --> make sure we can SSH into the instance

-when lauch we need a key pair that allows us to SSH into the instance we just launched
-in this case we create a new key pair
-once instance is going, if it's not free, billing starts
-can right click--> access instance state--> start/stop/terminate

### SSH 
-SSH works for Mac, Linux, and newer windows operating systems 
-SHH is a function--> control a remote machine/server using a CL 
-we have our EC2 machine via AWS with a public IP address and port 22--> want to access it with my machine--> we will access via CLI and port 22 
-In instance console--> public IP, public DNS--> how we can connect over the web! --> copy the public IP address--> click on security group--> inbound rules--> check tcp protocol 
-open a terminal 
-"ssh ec2-user@" followed by IP --> see that permission is denied --> because we need to use the key pair file that downloaded immediately after I launched the instance 
-**"AWS course Tess$ ssh -i EC2tutorial.pem ec2-user@[IP address]"**
-get a warning about unprotected private key file --> permission is 0644 --> private key can link --> will say bad permissions
-to fix: "chmod 0400 EC2tutorial.pem" and then rerun "ssh -i EC2tutorial.pem ec2-user@[IP address]" in CL --> NOW have SSH'd into the Amazon Linux 2 AMI, we're in the machine 
-we will run lots of commands from our EC2 machine/server! 
-to exit: control-d or write "exit" 

### EC2 Instance Connect (alternative to SSH from the CLI)
-go to EC2 console on AWS --> click connect button next to launch instance --> see options for how to connect and select EC2 instance connection, a browser-based instance connection --> ec2-user --> click connect --> in! 
-from the browser we can do the same sort of thing!!no terminal and no keys--> AWS handles keys behind the scenes 
-STILL need SSH port 22 rule in instance for this to work
-only works with Amazon Linux 2 AMI!

### 





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
HOW TO:
-In instance console--> public IP, public DNS--> how we can connect over the web! --> copy the public IP address--> click on security group--> inbound rules--> check tcp protocol 
-open a terminal 
-"ssh ec2-user@" followed by IP --> see that permission is denied --> because we need to use the key pair file that downloaded immediately after I launched the instance 
-**"AWS course Tess$ ssh -i EC2tutorial.pem ec2-user@[IP address]"**
-get a warning about unprotected private key file --> permission is 0644 --> private key can link --> will say bad permissions
-to fix: "chmod 0400 EC2tutorial.pem" and then rerun "ssh -i EC2tutorial.pem ec2-user@[IP address]" in CL --> NOW have SSH'd into the Amazon Linux 2 AMI, we're in the machine 
-we will run lots of commands from our EC2 machine/server! 
-to exit: control-d or write "exit" 

### EC2 Instance Connect (alternative to SSH from the CLI)
HOW TO:
-go to EC2 console on AWS --> click connect button next to launch instance --> see options for how to connect and select EC2 instance connection, a browser-based instance connection --> ec2-user --> click connect --> in! 
-from the browser we can do the same sort of thing!!no terminal and no keys--> AWS handles keys behind the scenes 
-STILL need SSH port 22 rule in instance for this to work
-only works with Amazon Linux 2 AMI!

### Intro to Security Groups
-fundamental of AWS network security--> control how traffic will be allowed into my EC2 machines
-controls inbound and outbound traffic 
-big part of amazon cloud 
-use "allow", "inbound", "outbound" ports 
-go to security group page to see the rules for what traffic is allowed in or out 
-inbound has our SSH rule
-default all traffic is allowed outbound

-if delete the inbound SSH rule--> not allowed anything to go through port 22 and will have a timeout when you try to SSH

-click edit --> add custom TCP rule--> to add back the inbound SSH rule! 
-ANYTIME you get a timeout on any machine, on any port--> Most likely a security group issue!

### Deeper Dive into Security Groups 
-act as firewalls for EC2 instances - these groups live outside the EC2!!
-regulate access to ports, authorized IP ranges 

-security groups can be attached to multiple instances
-an instance can also have multiple security groups 
-security groups are locked to a region/VPC combo

-good practice to have one separate security group specfically for SSH access!! since SSH is more complicated 

-if NO timeout and there is a "connected refused" error then the security group worked! the traffic went through the security group and the app itself has an error 

-default: all inbound traffic blocked, all outbound traffic allowed

HOW TO: reference security groups from other security groups 
-attach security groups according to how you want the instances to communicate
-why? we can allow instances to connect straight through via matching security groups--> directionality of connectivity is determined this way (a lot like a very simple neural network!)

### Private vs Public IPv4)
-issue of security 
-neworking --> 2 sorts of IP--> IPv4 and IPV3
-IPv4 --> 4 #s separated by 3 dots --> most common 
-IPv6 --> less common --> long strange of numbers and then letters --> more for IoT
-we will use only IPv4 in this course but AWS supports IPv6 too!
-IPv4 --> 3.7 billion addresses in public space

-public server--> public IP
-private network (like a company) --> private IP range --> all computers in the private network can talk to one another using the private IP 
-the private network can also talk to other public networks
-public IP unique across whole internet, can be access across the whole internet--> public IPs can be geolocate easily 
VS
-private IP can only be identified on the private network BUT two companies can have the SAME IP
-private network machines usually connect to interet via internet gateway (a proxy)
-only a specified range of IPs can be used as private IPs 

Elastic IPs --> when you start/stop Ec2 instance it can change it's public IP!!--> if we need a fixed public IP for the instance--> need something called an Elastic IP address --> a public IPv4 IP can own as long as it is not deleted --> can attach it to only one instance at a time 
-can mask the failure of an instance or software by rapidly remapping the address to another instance in your account 
-can only have 5 elastic IPs in the AWS account
-overall, try to avoid using, poor architectural choice
INSTEAD:
-use random public IP and register a DNS name to it! 
OR
-load balancer and no public IP at all (see later notes) --> best pattern for AWS

default: private IP on EC2 machine for internal AWS network AND a public IP for the internet 

-when SSH into EC2 machines can't use private IP because we're not on the same network (unless have a VPN) --> can only use public but if my computer is stopped / started the public IP can change! 

### Private vs. Public vs. Elastic IP Hands On 

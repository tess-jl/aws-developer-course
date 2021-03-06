# Section 3: AWS Fundamentals: IAM + EC2 

### IAM Service overview
-AWS consoles are region-scoped (excpt IAM and S3)
-root account should never be used (and shared)
-MFA
-least privilege principle

-identity and access managemnet
1. users
1. groups = groups of users
1. roles  = what we give to machines
1. policies = JSON docs that define what users, groups, roles can do

-Big enterprises use IAM Federation (when a company uses AWS)--> one can log in with company credentials --> identity feds use the SAML (security assertion markup language) standard(active directory)

1 IAM user / physical person 
1 IAM role / app
IAM credentials KEPT SECRET- NEVER WRITE IN CODE!
Never use root account except for initial setup--> never use ROOT IAM CREDENTIALS! 

### EC2 Service
-EC2 consists of:
1. renting virtual machines (EC2)
1. Storing data on virtual drives (EBS)
1. Distributing load across machines (ELB)
1. Scaling the services using an auto-scaling group (ASG)

-when start an instance have to have its operating system on **EBS volume**
-when launch an instance can have tags (key/value pairs which allow you to identify instance and classify it)

### SSH 
-we have our EC2 machine via AWS with a public IP address and port 22 

 --> copy the public IP address--> click on security group--> inbound rules--> check tcp protocol 

-"ssh ec2-user@" followed by IP --> see that permission is denied --> because we *need to use the key pair file that downloaded immediately after I launched the instance* 
-**"AWS course Tess$ ssh -i EC2tutorial.pem ec2-user@[IP address]"**
-get a warning about unprotected private key file --> permission is 0644 --> private key can link --> will say bad permissions --> fix: "chmod 0400 EC2tutorial.pem" and then rerun "ssh -i EC2tutorial.pem ec2-user@[IP address]" in CL --> NOW have SSH'd into the Amazon Linux 2 AMI, we're in the machine 


### Security Groups
**security group** = firewall around the instance --> make sure we can SSH into the instance
-**default: all inbound traffic blocked, all outbound traffic allowed**
-fundamental of AWS network security--> **act as a firewall to control how traffic will be allowed into my EC2 machines**
-controls inbound and outbound traffic --> regulate access to ports, authorized IP ranges 
-use "allow", "inbound", "outbound" ports 
-inbound has our SSH rule
-default all traffic is allowed outbound

-if delete the inbound SSH rule--> not allowed anything to go through port 22 and will have a timeout when you try to SSH
-click edit --> add custom TCP rule--> to add back the inbound SSH rule! 
-*ANYTIME you get a timeout on any machine, on any port--> Most likely a security group issue!*


-security groups can be attached to multiple instances
-an instance can also have multiple security groups 
-*security groups are locked to a region/VPC combo*

-good practice to have one separate security group specfically for SSH access! since SSH is more complicated 

-*if NO timeout and there is a "connected refused" error then the security group worked! the traffic went through the security group and the app itself has an error* 

HOW TO: *reference security groups from other security groups* 
-attach security groups according to how you want the instances to communicate
-why? we can allow instances to connect straight through via matching security groups--> directionality of connectivity is determined this way (a lot like a very simple neural network!)

### Private vs Public IPv4
-neworking --> 2 sorts of IP--> IPv4 and IPV3
-IPv4 --> 4 #s separated by 3 dots --> most common 
-IPv6 --> less common --> long strange of numbers and then letters --> more for IoT
-AWS supports both

-*public server--> public IP*
-private network (like a company) --> private IP range --> all computers in the private network can talk to one another using the private IP 
-the private network can also talk to other public networks
-public IP unique across whole internet, can be access across the whole internet--> public IPs can be geolocate easily 
VS
-private IP can only be identified on the private network BUT two companies can have the SAME IP
-private network machines usually connect to interet via internet gateway (a proxy)
-only a specified range of IPs can be used as private IPs 

**Elastic IPs** --> when you start/stop EC2 instance it can change it's public IP!!--> if we need a fixed public IP for the instance--> need something called an **Elastic IP address** = a public IPv4 IP can own as long as it is not deleted --> can attach it to only one instance at a time 
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
-been using a public IP to SSH 
-*private IP cannot be used to SSH*--> b/c not in the same network as EC2 instance 
-when we stop the EC2 instance--> public IP not shown at all--> private is the same --> restart the instance --> public IP has changed therefore need to update the SSH command

HOW TO: Connect our EC2 instance to elastic IP
-elastic IPs in the bottom left--> allocate a new address--> IP is not attached to anything --> right click and associate the elastic IP with the instance that is running--> now the public IP listed on the instance is our elastic IP
-this elastic IPv4 will remain even if the instance is stopped!


### EC2 User Data
-possible to bootstrap our instances using an EC2 User data script
-bootstrapping = launching commands when the machine starts 
-EC2 user data made to automate boot tasks (e.g. installing updates, software, whatever!)
-these scripts run with the root user--> any command will have the sudo rights 

-want this EC2 instance to have an Apache HTTP server installed on it--> so that we can display a web page --> going to use a user data script for this 
-script will be executed when the instance first boots 

-terminate instance
-launch a new instance--> under "configure instance" tab look at Advanced Details and find "User data"
-paste a script to automate 
```
#!/bin/bash
# install httpd (Linux 2 version)
yum update -y
yum install -y httpd.x86_64
systemctl start httpd.service
systemctl enable httpd.service
echo "Hello World from $(hostname -f)" > /var/www/html/index.html"
```
keep moving until "Configure Security Group" tab--> select the existing security group i've already made 

--> can NOW go to the public IP and see that the script was run because the boot time script launched the web page
-can ssh into this EC2 with the updated public IP address 
-to verify that all was done properly "cat /var/www/html/index.html" and we see the content of the webpage 

### EC2 Launch Types
-NEED to know all of them for the exam
1. **On Demand Instances** (short workload, predictable pricing)--> pay per second
1. **Reserved Instances** (long workloads) --> need a database for 1 or 3 years, much cheaper than On Demand, can select instance type, good for steady state
1. **Convertable Reserved Instances** (long workloads, flexible instances) --> can change EC2 instance type (slightly more expensive than Reserved)
1. **Schedule Reserved Instances** (only launch within window reserved, maybe only 1x/week)
1. **Spot Instances** (show workloads, cheap, can lose instances) --> 90% off, bid price, get unit as long as not much demand, can be lost when someone buys it--> good for any resilient workloads, NOT database
1. **Dedicated Instances** (no other customers will share hardware, most expensive) --> NOT full control over instance, might move between different hardware, can share hardware with another AWS customer with instances in same account
1. **Dedicated Hosts** (entire physical server dedicated to you --> full control over instance, visibility to sockets/cores, useful for BYOLntrol the instance placement, expensive)

### EC2 Good Things to Know / Checklist
-pricing = per hour 
https://aws.amazon.com/ec2/pricing/on-demand/

-What's an AMI ? Amazon Machine Image --> we've been using Amazon Linux 2 --> can be customized at runtime using EC2 User data
-can create a custom AMI --> for Linux or Windows machines --> may want to use to have faster boot time, pre-installed packages, security issues, etc. etc... 
-when build an AMI it is built for a specific AWS region!!

Instances have types--> 5 distinct characteristics
1. The RAM
1. The CPU
1. The I/O
1. The Network
1. The GPU

over 50 different instance types! https://ec2instances.info/ for more info... 
e.g. M = balanced, good at everything
T2/T3 instance types are burstable--> meaning that instance is OK but when it needs to do something suddenly the CPU can burst--> uses burst credits then has bad CPU --> see "Cloud Watch" to monitor the burst credit usage and load change in a GUI :) 

-CPU credits --> can look at how fast credits are accumulated
-T2 unlimited --> unlimited burst credits

### Need to know: 
* SSH into EC2 (and change .pem file permissions)
* Properly use security groups (right ports, right IPs)
* Fundamental differences between Private, Public, Elastic IPs
* how to use User Data to customize EC2 instance at boot time 
* that can build custom AMI to enhance OS

* EC2 instances are billed by the second and can be easily created and thrown away--> welcome to the cloud

___

IAM Users are defined on a global basis! but security groups are per region!

EC2 compute only paying when running 

CIDR Block = IPv6 related 

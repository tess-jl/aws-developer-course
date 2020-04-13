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
# Section 8: AWS Elastic Beanstalk 
-allows us to deploy applications in a way that isn't so manual--> easy, scalable, safe

### Elastic Beanstalk Overview
-as developer don't want to manage infastructure, just want to deploy code 
-only want to do configs for DBs, LBs, etc. 1x
-want auto scaling
-possibily want consistency across all environments 

**Elastic Beanstalk** leverages all the components we've seen before 
-managed service, config strategy and AWS takes care of rest
-has 3 architecture models 
* **single instance deployment** = good for dev 
* **LB and ASG** = good for prod or pre-prod web-apps
* **AGS only** = good for non-web apps in prod

Elastic Beanstalk has 3 components: 
1. App
1. App version (each deployment is assigned a version)
1. Environment name (dev, test, prod etc.)
--> deploy apps to envs and can promote app versions to the next gen if want 
-full control over lifecycle of environments:
1. create the app, create envs 
1. upload version + alias 
1. release to envs 

what can we deploy? 
-there is support for MANY platforms (most any language you'd use AND any Docker)
-can also write own custom platform (advanced)

## EB First Environment
-deleted EC2, DBs, LBs to prep 
-3 steps 
1. Select platform 
1. upload an app or use a sample 
1. run it 

--> when create app in beanstalk console and it is in process of deploying it--> says using S3 bucket for it (can see it in S3 console btw)
-when done deploying --> click events to see all of the events that auto happened --> bucket, security group, created in elastic IP, etc. 

-can click link on EB console and it gives links for what's next
-running in simpliest way--> dev mode therefore 1 EC2 --> if move to another env EC2 will update to multiple instances

left side menu--> logs--> request last 100 lines of log--> can see what happens on EC2 machine when app is deployed 
--> health = health of the app 
--> monitoring = how many CPUs etc. graphs 
--> manage updates 
--> events 
--> tags 

left hand menu for this app:
* envs
* app versions 
* saved configs 

on All Environments page --> can create a new env, revert to prior, swipe env URLs, or delete an app 
-each app can have many environments, each environment can be applied to an app version 

### EB Second Environment
on Environments page--> orange create environment button
* select web server env
* fill in form for new Node.js env--> click configure more options --> can select config presets as high availability (not free) --> can create a RDS via this --> BUT if created in EB then it will be delete with the EB too, sometimes better to do it externally and connect it 

-with high availability setup --> more components created! more security groups
-LB created for us auto 
-full-blown env created via EB 

### EB Deployment Modes 
-very popular question- which deployment mode is better for which situation?! 
recall:
* **single instance deployment** = good for dev b/c one EC2, one AZ, one LB
* **LB and ASG** = good for prod or pre-prod web-apps --> elastic LB, ASG spanning across multiple AZ, each instance own SGs (ELB will expose a DNS name which is wrapped by the elastic beanstalk DNS name)
* **AGS only** = good for non-web apps in prod

What about when we want to update a deployment?
Options for update: 
1. **All at once (deploy all in one go)** = fastest, but instances down temp 
1. **Rolling** = update a few instances at a time (bucket) --> once bucket healthy, move onto the next bucket
1. **Rolling with additional batches** = like rolling but spins up new instances to move the batch (therefore old app still available)
1. **Immutable** = spins up new instances in a new ASG, deplpus version to these instances and then swaps all instances when all is healthy 

**All at once (deploy all in one go)**
-good for quick iterations
-no additional cost 

**Rolling**
-app running below capacity 
-can set bucket size (i.e. capacity)
-app runs both versions simultaneously 
-no additional cost 
-could be a long deployment, tho 

**Rolling with additional batches**
-app running at (or sometimes over) capacity
-can set bucket size 
-running both simultaneously 
-small extra cost 
-additional batches removed at end 
-longer deployment 
-good for prod 

**Immutable** 
-zero downtime
-new code deployed to NEW instances on a temporary ASG 
-double capacity therefore high cost
-longest deployment
-quick rollback in case of failures (just terminate new ASG)
-great choice for prod 

**Blue/Green**
-not direct feature of EB 
-zero downtime and release facility 
-create a new "stage" env --> deploy v2 from there 
-new env = GREEN --> can be validated independently and rollback if issues 
-Route 53 an be setup using weighted policies to redirect a bit of traffic to the stage env to test everything
-when happy with the test env can just go ahead and swap URLs in EB so that test (GREEN) becomes the new env
--> very manual, not really embedded in EB

see AWS docs for summary on EB --> shows a nice table

### EB Deployment Hands On 
-see EB config--> deployment--> default = all at once but can choose any
-if select rolling --> can select 30 therefore 70% of fleet is always up 
-config updates (not asked about on exam)
-preferences

-can google EB sample app and download a Node.js sample app file --> update the sample for the redeployment--> zip it back up
-deploy for updates with immutable strategy--> events show us how temp ASG is created! etc. etc. --> new instance added to LB, once pass health check the new instances are then let off of temp ASG and moves instances to the original ASG 
-EB handles all the complexity for us!

### EB Advanced Concepts 
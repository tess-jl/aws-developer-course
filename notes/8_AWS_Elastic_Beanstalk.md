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

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
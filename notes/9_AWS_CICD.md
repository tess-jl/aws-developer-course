# Section 9: AWS CICD: CodeCommit, CodePipeline, CodeBuild, CodeDeploy 
-deploying apps automatically! push code and have it deployed automatically--> revolutionary!

### Intro to CICD in AWS 
-know how to use AWS manually
-know how to use AWS programmatically (CLI)
-know how to use AWS auto (via EB)
-instead we want to push to repo and have code deployed totally automatically! make sure only tested code pushed, try out dev, prod, etc. therore need CICD 

**Continuous Integratation Continuous Deployment (Delivery)** --> auto deployment 
PHASES:
**Code Commit** = store code 
**CodePipeline** = automating pipeline from code to EB 
**CodeBuild** = build and test code 
**CodeDeploy** = to deploy to EC2 fleets (not EB)

**Continuous Integration** = allows devs to push code to a code repo as often as posss (GutHub, CodeCommit, Bitbucket etc.)
* will have a test/build server that checks pushed code (CodeBuild, Jenkins, Travis, etc.)
* get feedback about tests that pass/fail from build server

**Continuous Delivery (or Deployment)** = ensure quick, predictable, reliable deployments --> therefore automated deployment
* can use CodeDeploy, Jenkins CD, Spinnaker
* moves from build server to deployment server --> deployment server runs scripts to make sure that the app is updated for every time we push

**Tech stack for CICD**
1. Code (AWS CodeCommit, GitHub, etc.)
1. Build and Test (AWS CodeBuild, Jenkins etc.)
1. Deploy and Provision (AWS EB, or sometimes own fleet of EC2s via CloudFormation deployed vy AWS CodeDeploy)
--> orchestration for all done by **AWS CodePipeline** on cloud

### CodeCommit Overview
**version control** = tracks changes
-repo can live on any machine but there is centralized repo online
**AWS CodeCommit** 
* private Git repos 
* no size limit 
* fully managed 
* code only in AWS cloud account 
* secure (ecrypted etc.)
* Integrated with 3rd party CI tools 

CodeCommit Security 
* interactions are done using git 
* **authentication** with Commit --> need SSH keys (AWS user can configure SSH keys in IAM console) OR can use HTTPS creds via AWS CLI Auth helper --> can also use MFA for extra security 
* **authorization** eith IAM policies manager user / roles rights to repos 
* **Encrytion** = repos are automatically encrypted at rest using KMS  --> encrypted in transit (via HTTPS or SSH) 
* **Cross Account access** NEVER share SSH keys! NEVER shar AWS creds --> instead use IAM Role in AWS account and use AWS STS (cross account AssumeRole API call)

CodeCommit vs GitHub 
-both git repos 
-both support PRs
-both integrated with CodeBuild
-both support HTTPS and SSH auth 
BUT
-Github has GitHub Users Security while CodeCommit uses AWS IAM roles and users 
-Github hosted by GitHub (and GitHub Enterprise hosted on your own servers) while CodeCommit is managed and hosted by AWS
-UI of GitHub is better 

**CodeCommit Notifications**
-can trigger notifs using **AWS SNS (Simple Notification Service)** or **AWS Lambda** or **AWS CloudWatch Event Rules**
* want SNS or Lambda when delete branches, trigger for pushes on master, notify external build, trigger lambda to preform code analysis (ANYTIME SOMETHING ADDED TO CODE)
* want to use CloudWatch Event Rules when issues with PRs --> trigger for PR updates, comments on code --> will trigger a notif into a thing called an **SNS topic** 

### CodeCommit Hands On part 1 
-part of the AWS dev tools for cloud
-create a repo 
-can connect to repo via HTTPS or SSH 
-commit index.html to master 
-can make a PR, can comment
-have a cool commit visualizer!
-can create a branch, tags
-SETTINGS:
* can rename repo --> gets ID, ARN 
* can setup **notifications** = events that users can get emails about repo events --> CloudWatch Event Rules sends into to SNS topic --> events are related to CODE MANAGEMENT 
* can also add **triggers** = events more related to people doing things to code to repo --> can have SNS be triggered or Lambda be triggered by these triggers

### CodeCommit Hands On part 2
-how to push code directly into codecommit
-go to IAM --> users --> myself --> security credentials --> see access keys
**SSH keys for AWS Code Commit** - can just upload keys here and copy SSH clone etc. 
**HTTPS Git credentials for AWS Code Commit** --> doing this in hands on:
* generate HTTPS keys
* go back to dev tools --> clone URL --> (make sure Git is installed on my laptop) --> the clone HTTPS URL from dev tools is copied --> 
```
git clone [url]
```
just like with the URL from github!
* enter username/password from the IAM credentials downloaded!
* use this just like git regularly except online host is CodeCommit

* remember both ways are supported, just used the HTTPS way

### CodePipeline Overview

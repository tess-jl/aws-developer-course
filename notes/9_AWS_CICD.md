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
-visual tool for continuous delivery 
-integrates **sources** (GitHub, CodeCommit, Amazon S3) with **builds** (CodeBuild, Travis, etc.)
-also does load testing with 3rd party tools 
-also does deploy (AWS CodeDeploy, EB, CloudFormation, ECS, etc.)
STAGES
* each with sequential and/or parallel 
* manual approval can be defined at any state 

**artifacts** = created by each stage, files that are passed and stored via S3 and passed onto next stage

troubleshooting 
* **state change in pipeline** generates a CloudWatch Event rule --> return SNS notifcations --> I can create events for whatever I want to do troubleshooting
* if pipeline fails a stage or doesn't deploy anything --> get info from console
* can audit API calls to CloudWatch via **CloudTrial** (used to audit ANY API calls across AWS)
* if pipeline can't preform an action make sure IAM Service Role attached has right permissions (Policy)

### CodePipeline Hands On 
-create pipeline--> name it--> create service role (IAM role) with right permissions (req.)
-choose source (codeCommit repo)
-choose build (optional)
-choose deploy stage (can be many things)--> we chose EB

-now running a new EB version via the pipeline
-note, app need to have a package.json or equivalent to be deployed 
-can add state to deploy to prod stage --> add action group "Manual approval" , can specify SNS topic etc. can add ANOTHER action groups 
-Stages have multiple **action groups** 
-this next stage is the prod deployment as seen in the pipeline
-code change is propogated across pipeline!

### CodeBuild Overview
-used for building/testing --> provide commands to server and server does something for both 

OVERVIEW
-fully managed service 
-alternative to tools like Jenkins
-continuous scaling (no build queue)
-pay for usage of CodeBuild, not if have it or not (1 min/day then pay for that one min)
-leverages Docker under the hood for reproducible builds
-possible to extend capabilities leveraging our own base Docker images 
-secure(integration with KMS = for build artifacts, IAM = for build permissions, VPC = for network security, CloudTRail = for API call logging)

What does it do? 
* can source code from different places (i.e. Github, CodeCommit, S3 etc.)
* **buildspec.yml** file in the code = build instructions
* outputs logs to S3 and AWS CloudWatch 
* metrics to monitor CodeBuild statistics
* can use CloudWatch Alarms to detect failed builds and trigger notifications
* CloudWatch Events / Lambda as glue 
* SNS notifs 
* can reproduce CodeBuild locally to troubleshoot 
* builds defined within CodePipeline or CodeBuild itself 

What can it build? 
* most code --> supports many languages! 
* can also use Docker to extend any environment you like 

how it works? 
1. source code with buildspec.yml at root AND build Docker Image 
1. code build triggered --> **CodeBuild Container** starts on Docker Image, uses run instructions from the buildspec.yml 
1. (OPTIONAL) AWS S3 cache bucket to cache dependencies, artifacts, etc. to increase preformance 
1. If build passes --> output sent to S3 bucket (artifacts bucket) --> artifact then to S3 bucket 
1. if build passes --> logs saved to CloudWatch or S3 

**buildspec.yml**
* must be at root 
* define env vars (plaintext OR **SSM parameter store**) 
* phases (commands to run) --> (1) **install** = install parameters for build, (2) **pre-build** = final commands to execute before build, (3) **build** = actual build commands, (4) **post-build** = finishing touches like .zip creation 
* **artifacts** are then defined --> uploaded to S3 and encrypted with KMS 
* cache to S3 defined --> usually dependencies 

**local build** = can be done on CodeBuild, done for troubleshooting beyond logs
* can be run on own computer but need to install Docker --> leverge **CodeBuild Agent** 

### CodeBuild Hands on part 1
-e.g. want to test that 
-choose source (in our case CodeCommit, but many other AWS services abailable)
-codebuild runs a docker image that will run our test --> 2 kinds of Docker Image: 
1. Managed Image (by AWS CodeBuild)
1. Custom Image (perhaps more specific for company)
-env--> OP = unbuntu, standard, latest image, type=Linux 
-service role--> new service role auto creates a new IAM service role for us 
-some extra config (optional, lots of stuff including env vars)

-MOST important = **build spec** = build specification, a file that tells code build how to build our project
* use a buildspec file --> by default will look in root 

-artifacts
-can enable cache (S3 or local)
-create build

-try build with code as-is (no spec!) --> fails as expected

### CodeBuild Hands on part 2
-creating file in browser
```
version: 0.2

phases: 
    install:
        runtime-versions:
            nodejs: 10
        commands:
            - echo "installing something"
    pre_build:
        commands: 
            - echo "we are in the pre build phase"
    build:
        commands:
            - echo "we are in the build block"
            - echo "we will run some tests"
            - grep -Fq "Congratulations" index.html
    post_build:
        commands:
            - echo "we are in the post build phase"
```
yml file = key/value pair file--> note all of the sequential phases 

-re-run build --> can look at **phase details** to show ALL the steps and how long each phase takes! 
-**build log** can click to see all of the logs from CloudWatch logs--> see the echos from the buildspec.yml 

How do we integrate this build project into our code deploy? 
-edit pipeline to include a testing stage between Source and Deploy to dev --> see BuildAndTest stage added with source as AWS CodeBuild

-can edit out the "congratulations" in index.html to see the test we made fail! 
-source changes
-build fails --> tells us why! THEREFORE EB does NOT have the bad code we changed
--> change code back to including the congratulations and now it does work! 
* see build history for entire history of builds!

### CodeDeploy Overview 
-most obscure 
-want to deploy app auto to many EC2 instances (not managed by EB)
-can do this deploy with open source tools (ansible, terraform, cheft, Puppet) --> CodeDeploy is the AWS in-house version, managed service

How to make it work? 
* Each EC2 Machine (or On Promise machine) MUST be running the **CodeDeploy Agent** 
* agent is continously polling CodeDeploy for work to do 
* CodeDeploy sends **appspec.yml** file 
* app pulled from GitHub or S3 
* EC2 will run the deployment instructions from appsepc.yml 
* agent will report if there was success/failure on instance 
--> have source code with appspec.yml file at root --> push code to Github or S3 --> triggered a deployment --> agent polling sees the trigger --> code downloaded onto EC2 instances and agent will run what is on appspec.yml to deploy code correctly 

Other things to know: 
* EC2s are grouped by deployment group (dev, prod, etc.)
* lots of flexibility to define what kind of deployments (more flexible than EB, more complex too)
* can integrate with CodePipeline!
* can re-use existing sets tools, any kind of app, auto scaling integration 
* can do blue/green but only works with EC2 instances (not on premise)
* support for lambda deployments
* only deploys, does not provision resources (i.e. assumes EC2s exist already)

CodeDeploy Primary Components
* **application** (with unqiue name)
* **compute platform** = EC2/Om-Premise or Lambda
* **Deployment configuration** = rules for success or failure (i.e. for EC2/On-Premise can specify min num of healthy instances, for lambda how want traffic routed)
* **Deployment group** = group of tagged instances (allows gradual deploy)
* **Deployment type** = in-place deployment or blue/green deploymenet 
* **IAM instance profile** --> need to give EC2 permissions to pull from S3/ GitHub 
* **Application revision** = app's code and the appspec.yml file 
* **service role** = role for CodeDeploy to perform what it needs 
* **Target revision** = app version of target deployment 

**appspec.yml**
-2 sections: 
* **file section** = how to source and copy from S3 / GitHub to filesystem 
* **Hooks** = instructions on how to deploy the new version (can have timeouts), order of hook events is: 
1. ApplicationStop 
1. DownloadBundle 
1. BeforeInstall --> then app installs
1. AfterInstall
1. ApplicationStart
1. **ValidateService** = like a health check

CodeDeploy deployment config 
* can deploy to one EC2 at a time, if one fails deployment stops 
* half at a time 
* all at once (good for dev)
* custom
-with failures instances stay in failed state and new deployments will first be deployed to failed state instances 
-to rollback--> redeploy old deployment or enable auto rollback for failures
-**deployment targets** can be set of EC2 instances with tags OR deploy directly to ASG OR mix with deployment segments OR customization via scripts (**DEPLOYMENT_GROUP_NAME** env var)

### CodeDeploy Hands On 
-need to create 2 new IAM roles--> one for CodeDeploy another for EC2 instance 

-then create a CodeDeploy 
-create a deployment group BUT first need to make sure we have our EC2 instances for it 
-set up new EC2 instance and for IAM role attach the EC2InstanceForCodeDeploy we just made
-launch EC2

-ssh into the new EC2
-install CodeDeploy Agent on this EC2!!
```
# Installing CodeDeploy Agent
sudo yum update
sudo yum install ruby

# Download the agent (replace the region)
wget https://aws-codedeploy-eu-west-3.s3.eu-west-3.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
sudo service codedeploy-agent status
```
after running all those commands on EC2: 
-back to CodeDeploy --> create **deployment group** --> can be sets of EC2 instances
-first need to tag our EC2 instances!
-then go back to deployment group--> name it --> select IAM role --> select deployment type (in place for this) --> env configs (use the tag from EC2 we just set!) --> deployment settings --> don't need a LB for now 
-create deployment group 
-ready to do first CodeDeploy deployments!

-go back to Application in dev tools 
-create deployment 
-select deployment group 

-create S3 bucket for this deployment! deploying a sample linux app (see SampleApp_Linux in code dir) with the appspec.yml (files and hook)
-upload .zip of that app to the bucket 
-copy path 

back in CodeDeploy: 
revision location = path copied from bucket 
-deploy 

-can check the public IP and get the congrats message

### AWS CodeStar
-what is diff between and CodeCommit? 
**CodeStar** = integrated solution that regroups things like GitHub, CodeCommit, CodeBuild etc.. CICD services 
-wrapper around everything--> one dashboard
-helps quickly create CICD-ready projects for EC2, Lambda, Beanstalk 
-supports many languages 
-issue-tracking integration 
-ability to integrate with Cloud9
-one dashboard for all components one stop for all management
-free service (only pay for underlying usage of services)
-limited customization 

Hands on
-go to CodeStar --> select Node --> name it --> create project--> select key pair 
-pick how to edit code 
-dashboard shows nice viz! 
-codestar is linked to all underlying services
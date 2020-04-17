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
# Section 16: AWS Serverless: SAM (Serverless Application Model)
-how we do manage our serverless app? could use cloudformation but it's complicated--> instead, SAM 
-SAM = shortcut to cloudformation (YAML files)

### SAM Overview
-**SAM (Serverless Application Model)** = serverless development framework used to write and deploy serverless apps
* all config = YAML CloudFormation code
* supports anything that CloudFormation uses (outputs, mappings, params, resources, etc.)
* 2 commands to deploy to AWS 
* SAM can use CodeDeploy to deploy Lambda functions
* SAM helps us run Lambda, API Gateway, DynamoDB locally (therefore don't need to deploy lambda function to test)

SAM Recipe 
* **Transform Header** = what indicates it's a SAM template --> e.g. Transform: Serverless-2018-10-31
--> this transform header **indicates how this SAM file should be transformed into much more complex cloudformation YAML** 
* 3 resources see: 
1. AWS::Serverless::Function --> Lambda
1. AWS::Serverless::Api --> API Gateway
1. AWS::Serverless::SimpleTable --> DynamoDB
* package with cloudfomration package or **sam package**
* deploy with cloudformation deploy or **sam deploy** 

**change set** = how cloudformation should take the existing state and move it to next state based on modifications

### Installing the SAM CLI 
https://github.com/awslabs/aws-sam-cli
can install a nice CLI to develop locally via aws-sam-cli --> see corresponding GitHub 
* need Docker, Python, AWS CLI 
* use python to install CLI 

### Creating First SAM Project
```
sam init
```
creates a lot of files for us but need to specify a lot 

Instead can do it file by file: 
look at examples: https://github.com/aws-samples/serverless-app-examples 
-need tample.yaml, app.py, and commands.sh 

### Deploying SAM Project 

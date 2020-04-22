# Section 10: AWS CloudFormation 
-concept = infastructure as code --> done via CloudFormation (been using this when use EB)

### CloudFormation Overview
-so far work has been largely manual --> would have to manually reproduce everything if we were in another region, another AWS account, or if everything was deleted
-SO we need infastructure to be code **infastructure as code**

**CloudFormation** = declarative way to outlining AWS infastructure for any resources 
-e.g. create a template with security group, 2 EĆ2s, LB, etc. 
-CloudFormation creates all things with right configs

BENEFITS
-infastructure as code
-version controlled 
-code reviewable 
-free but each stack tagged with **identifier** so can see cost of stack's resources --> can estimate cost of these 
-can implement deletion of templates for certain times to save money 
-productivity- can destroy and re-create infastructure 
-separation of concern--> many stacks for many apps, e.g. VPC stacks, network stacks, app stacks
-can leverage existing templates

How does CloudFormation work? 
-have to upload templates in S3 --> referenced in CloudFormation 
-to update template need to re-upload a new version of the template, can't edit existing one!
-stacks identified by name 
-if delete stack all associated resources will be deleted too 

Deploying templates: 
-**manual** = editing templates in CloudFormation Designer, use console to input parameters etc. 
-**automated** = editing templates in YAML file, use AWS CLI to deploy

Building Blocks
**Template components** 
1. **Resources** = aws resources declared in template (MANDATORY), e.g. EC2s, LBS, etc
1. **Parametrs** = dynamic inputs for template 
1. **Mappings** = static vars for template 
1. **Outputs** = references to what has been created in template
1. **Conditionals** = if statements that control what resources are created under certain conditions 
1. **Metadata** 

**template helpers** 
1. **References**
1. **Functions** 

exam does not require us to write cloudFormation but I am expected to read cloud formation, what features should be used, etc. 
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

### Create Stack Hands On 
-create EC2
-add elastic IP to it 
-add 2 security groups 

-since cloudformation is the backbone of EB there should already be stacks in our cloudformation console --> one for each EB env 

cloudformation Designer for viz 

-create a stack 
-can select template for CloudFormation
-using template 0-just-ec2.yaml --> keeping with all default settings
-can use the price estimation tool 
-create stack 
-most important tab in stack console is the template 

### Update and Delete Stack Hands On 
-remember that have to provide an entirely new cloudformation template to actually update an existing one 
-using 1-ec2-with-sg-eip.yml 

-at stack --> action --> update stack 
-cannot change name of already-created cloudFormation template!

-check replacement true column--> code handles infastructure and implementation of making the infastructure! 
-all aspects of the template are re-added and previous resources deleted
-resources are prefixed by stack name

-if want to delete go to actions --> delete --> resources deleted in the right order too! 
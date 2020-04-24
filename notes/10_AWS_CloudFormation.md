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

### YAML 
-used all across AWS 
-scripting language, data language --> like JSON --> both can be used on CloudFormation
-JSON is horrible for cloud formation tho 

-a lot of key/value pairs, objects, also:
* nested objects
* arrays aka lists (see - )
* multiline string (see | )
* comments 

### CloudFormation Resources (components)
-core of cloudformation template, mandatory 
-represent the different AWS components 
-are declared 
-can reference each other 
-AWS knows how to create them in cloudformation 
-224+ resources! **AWS: :aws-product-name: :data-type name**

documentation online at AWS
-can find JSON and YAML forms for each resource, shows what is customizable 
-docs show up how to write the template for CloudFormation
-everything that can be configured through UI can be written as code for cloudformation template
-all of the code goes under the YAML block **Resources**

FAQs
can create dynamic amount of resource? No- everything has to be declared, can't preform code generatation 

is every AWS service support? Almost, only a few that are not (work around AWS Lambda Custom Resources)

### CloudFormation Parameters 
**Parameters** = way to provide inputs to template 
-important for reusing templates across the company, accounts, region 
-some inputs cannot be determined ahead of time
-powerful, controlled, can prevent error from happening in template due to types

When do I use? 
-if CloudFormation resource config is likely to change in the future 
-parameters mean we don't have to reupload a template to change content


Settings
* **type** --> data type, either string, commadelimitedList, List<type>, AWS param
* **description** --> constraints, constraintDescription (string), min/max length for string, min/max value for number, defaults, allowedValues (array), AllowedPattern(regex), NoEcho(Boolean)

How do we reference a parameter? 
-need to use the function "Ref" **Fn::Ref**
-shorthand for this in YAML = **!Ref**
-can be used to reference other elements in the template

**Pseudo Parameters**
-AWS-offered params we can use 
-enabled by default 
-can use at any time
e.g. AWS::AccountId, AWS::Region, etc.

### CloudFormation Mapping 
**Mappings** = fixed variables within template, need to be hard coded
-good for differentiating between envs (dev vs prod), regions, AMI types, etc. 
-all under a Mappings section in YMAL 

when will we use mappings vs params? 
-mappings when know all values that can be deduced from variables
-safer control of templates
IF values needs to be user-specific and don't know exactly what it will be in advance then use params 

**Fn::FindInMap** aka **!FindInMap** 
-needs **[ MapName, TopLevelKey, SecondLevelKey ]**
-returns named value from a specific key 

### CloudFormation Outputs
-declares optional **outputs** values that can be imported into other stacks (if exported first)
-for linking templates, therefore collaboration across stacks
e.g. network CloudFormation template and output vars are VPC ID and subnets ID
**CANNOT delete CloudFormation Stack if outputs are referenced by another stack!**

**Outputs:** in YAML
-Export block is optional! for other stacks to import --> via **FN::ImportValue** or **!ImportValue**
CROSS STACK REFERENCE

### CloudFormation Conditions 
-based on a **condition**, the creation of resources or outputs is controlled
e.g. to control if in __ env, don't create ___
-each condition can reference another condition, parameter value, or mapping 

**Conditions:** block
-see !Equals implemented in example
-logical functions used: 
* **!Equals**
* **!And**
* **!If**
* **!Not**
* **!Or** 

when using the condition:
-in Resources, e.g.--> Condition: [nameOfCondition] --> only created resource if condition is true

### CloudFormation Intrinsic Functions
Must know for exam: 
* **!Ref** --> returns val of param, returns ID of resource
* **!GetAtt** --> returns list of attributes of a resource
* **!FindInMap** --> returns named value from a specific key (need to specify MapName, TopLevelKey and SecondLevelKey)
* **!ImportValue** --> returns value exported from another template (need to give it name)
* **!Join** --> returns joined values (need to give delimiter name and comma-delimited list of values as args)
* **!Sub** --> aka Substitue --> allows to subsitute vars within strings, customize templates --> returns subsituted value (string must contain ${VarName} syntax for subsitution)
* **!Equals**
* **!And**
* **!If**
* **!Not**
* **!Or** 

### CloudFormation Rollbacks
-if **stack creation fails** --> default = everything just rolls back (gets deleted), can look at log 
-also option to disable rollback and troubleshoot what happened 
-if **stack update fails** --> auto rollback to previous known working state 
-can see in log what happened via error messages

see "UPDATE_FAILED" error message that tells us exactly why and then update rollback goes into progress--> all resources that were created in update are then deleted with the failure of the update
-therefore CloudFormation is quite safe

-cannot update a stack once it has failed in creation (via update, etc.)
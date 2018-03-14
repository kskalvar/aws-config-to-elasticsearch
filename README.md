AWS Config Snapshots to AWS Elasticsearch
===================================================

This solution shows how to automate the ingestion of your AWS Config snapshots into the ElasticSearch/Logstash/Kibana (ELK) stack for searching and mapping your AWS environments.  This readme updates an article "How to Analyze AWS Config Snapshots with ElasticSearch and Kibana by Vladimir Budilov" referenced below and provides a more basic step by step process.

## Configure AWS Config Service

Use the AWS Console to configure the AWS Config Service.  This is a step by step process.

### AWS Config Console
Get started

#### Settings
Resource types to record  
```
All resources: checked
```  

Amazon S3 bucket  
```
Create a bucket: selected
```

AWS Config role  
```
Create a role: selected
```  
Next

#### AWS Config rules
Next

#### Review
Confirm  

### AWS Resources Created
S3 Bucket  
```
config-bucket-<AWS Account Id>
```   
IAM Roles  
```
config-role-us-east-1
```    
IAM Policies  
```
Customer managed:
config-role-us-east-1_AWSConfigDeliveryPermissions_us-east-1
```  

## Configure AWS Elasticsearch Service

Use the AWS Console to configure the AWS Elasticsearch Service. This is a step by step process.

### AWS Elasticsearch Console
Create a new domain

#### Define domain
Elasticsearch domain name:  
```
aws-config-es
```    
Elasticsearch version:  
```
5.5
```  
Next

#### Configure cluster
Instance count:  
```
1
```  
Instance type:  
```
t2.small.elasticsearch
```  
Next

#### Set up access
Network configuration  
```
Public access
```  
Access policy  
Select a template pull-down  
```
Allow open access to the domain
Note:  acknowledge risk pop-up
```  
Next

#### Review
Confirm

#### AWS Elasticsearch Resources Created

From the AWS Elasticsearch Console you will see ```aws-config-es``` domain configuration status.  "Configuration state” shows the status of the just created domain and initially will display “Loading”.  Once it shows “Active” the domain will be ready to access.  Node configuration took 12 minutes to become active for this example.  

Drill down to the ```aws-config-es``` domain and select the "Overview" tab.  

Of specific interest for this task are:  

```
"Endpoint: <Endpoint URL>"
"Kibana: <Kibana URL>"
```

## Configure AWS EC2 Instance

Use the AWS Console to configure the AWS EC2 Instance. The EC2 Instance will be used to run the python script which will import ASWS Config Snapshots into Elasticsearch.  This is a step by step process.

### AWS EC2 Console
Instances/Launch Instance

#### Choose AMI
```
Amazon Linux AMI 2017.09.1 (HVM), SSD Volume Type - ami-97785bed
```  
Select

#### Choose Instance Type
```
t2.micro
```
Next

#### Configure Instance
Shutdown behavior  
```
Terminate
```
Next

#### Add Storage
Next

#### Add Tags
Next

#### Configure Security Group
Select an existing security group
```
Use default
```
Next

#### Review and Launch
Launch  
Select an existing key pair or create a new key pair  
```
Note:  acknowledge private key checkbox
```
Launch Instance  
View instances

### Configure AMAZON AMI instance

You will need to ssh into your EC2 instance and complete the following configuration:

```
aws configure

sudo yum update
sudo yum install git –y

git clone https://github.com/awslabs/aws-config-to-elasticsearch.git

cd aws-config-to-elasticsearch

Modify requirements.txt 
   boto3
   botocore
   requests
   gzip-reader
   simplejson
   argparse

sudo pip install -r ./requirements.txt
```

### Run Import Script from AWS EC2 Instance

```
Using <Endpoint URL> from "AWS Elasticsearch Resources Created"

cd aws-config-to-elasticsearch/aws_config_to_es
./esingest.py --destination <Endpoint URL>:80 --verbose
```

## Configure Kibana

Use the Kabana Console to configure the index pattern and time filter.  This is a step by step process.


### Access Kabana via Web Browser 
```
Using <Kibana URL> from "AWS Elasticsearch Resources Created"
```

#### Management View
Index name or pattern
```
*
```
Time Filter field name
```
snapshotTimeIso
```
Create

##### Discover View
Go to the Discover View to browse data


## References
How to Analyze AWS Config Snapshots with Elasticsearch and Kibana  
https://aws.amazon.com/blogs/developer/how-to-analyze-aws-config-snapshots-with-elasticsearch-and-kibana/

Import your AWS Config Snapshots into Elasticsearch  
https://github.com/awslabs/aws-config-to-elasticsearch

Understanding the basic components of AWS Config will help you get the most out of this service  
https://docs.aws.amazon.com/config/latest/developerguide/config-concepts.html

Creating and Configuring Amazon Elasticsearch Service Domains  
https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-createupdatedomains.html

Starting from Elasticsearch 6.0, all REST requests that include a body must also provide the correct content-type for that body  
https://www.elastic.co/blog/strict-content-type-checking-for-elasticsearch-rest-requests


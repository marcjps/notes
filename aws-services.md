# AWS Services

## Compute

Services for running code.

### EC2 

Run Virtual Machine instances in the cloud.

### ECS 

Run docker containers on your EC2 instances.  Create a ECS Cluster (which creates EC2 instances) then run a Task (one off) or a Service (continuous) on the cluster (using a docker image).  AWS specific (and simpler) version of Kubernetes.

### EKS

Managed Kubernetes.  Run docker containers on your EC2 instances.  Kubernetes is portable to other clouds or on-prem.  

### ECR

A registry to upload docker images to.  Tag images (e.g. live, staging, dev) and automatically delete old images.

### Lightsail

A tool for very easily setting up common applications on AWS.  (e.g. Drupal, Wordpress, etc)  Creates instances and all associated resources, and provides an simple management UI.

### Batch 

Do large computation work, by breaking the task into many jobs and executing them in batches.  Batch jobs must run on docker containers, using spot instances.


### Elastic Beanstalk

Creates application stacks, running a required platform (e.g. Java/.net/PHP etc).  Similar to Lightsail but instead of installing an application (Drupal), just installs the platform, for you to deploy an application.  Provides autoscaling, RDS, and deployment - provides a command line app to deploy application code.

### Lambda

Run serverless code.

### Serverless Application Repository

An area for publishing server less applications (including CloudFormation templates and source code), allowing you to browse and reuse them.  (similar to the marketplace)  It's really interesting to browse as you can review the code for all the serverless samples.


### Elastic Load Balancing

Incoming traffic load balancing service.

### VMWare Cloud on AWS

Creates a VMWare stack on the cloud, running the same tools as on-premises one.  Link to the on-premises one to easily move applications between locations.

## Storage

Services for storing data.

### S3

Store and host files in the cloud.  The first AWS service.

### Elastic Block Store

Manage fixed disk volumes for EC2 instances.

### Elastic File System

Attach dynamic shared volumes to EC2 instances.  It works like a share.

### Glacier

Long term cheap slow-retrieval storage (archiving).

### Storage Gateway

Allows on-premises services to use AWS storage services (e.g. S3).

### Snowball

Transfer big data to the cloud on a physical drive.

### Snowball Edge

?? data transfer

### Snowmobile

A snowball truck.

## Database

Services for hosting databases.

### Aurora

SQL relational database with MySQL compatibility.

### RDS

Host databases on instances.

### DynamoDB

NoSQL database. (similar to MongoDB)

### ElasticCache

Key/value store intended for caching.  (similar to memcached)

### Redshift

Database for Business Intelligence.

### Neptune

A graph database. (similar to Virtuoso)

### Database Migration Service

A tool for migrating databases from one platform to another (e.g. MS-SQL to MySQL)  It can do continuous replication.

## Migration

Services to help with moving existing data/services into the cloud.

### Migration Hub

### Discovery Service

### Database Migration Service

### Server Migration Service

## Networking & Content Delivery

### VPC

Manage virtual networks for EC2 instances.

### VPC PrivateLink

### CloudFront
 
A CDN cache. (similar to Akamai)

### Route 53

Managed DNS service, for internal and external DNS names.

### API Gateway

Create a REST API which sends requests to Lambda. (and other AWS services)  Also supports websockets.

### Direct Connect

A VPN between an AWS VPC and your premises.


## Developer Tools

Tools for developers.

### CodeStar

Helper that combines all the code management services below.

### CodeCommit

Source control.

### CodeBuild

Continuous integration. (similar to Jenkins?)

### CodeDeploy

Distribute builds to instances.

### CodePipeline

Probably automation of tasks after commits. (e.g. run tests)

### Cloud9

A cloud IDE.

### X-Ray

???

### Tools & SDKs

???

## Management Tools

Tools for managing the cloud account.

### CloudWatch

See analytics (e.g. traffic, performance, logs) from the other services.

### AutoScaling

### CloudFormation

Construct all types of cloud resources using scripts.

### CloudTrail
### Configs
### OpsWorks
### ServiceCatalog
### Systems Manager

A tool for managing servers at scale.  Session Manager provides remote console to EC2 instances from the browser, it keeps the console log for auditing.  Does not support RDP.

### Trusted Advisor
### Personal Health Dashboard
### Command Line Interface
### Management Console
### Managed Services

## Media Services

Work with video and audio.

### Elastic Transcoder
### Kinesis Video Streams
### Elemental MediaConvert
### Elemental MediaLive
### Elemental MediaPackage
### Elemental MediaStore
### Elemental MediaTailor

## Machine Learning

### SageMaker

Model training.

### Comprehend
### Lex

Build chatbots.

### Polly
### Rekognition

Identify items in images.

### Machine Learning
### Translate

Translate into different languages.

### Transcribe

Transcribe text from audio or video.

### DeepLens
### Deep Learning AMIs
### Apache MXnet
### Tensorflow on AWS

## Analytics

Data cleanup and analysis.

### Athena
### EMR
### CloudSearch
### ElasticSearch Service
### Kinesis
### Redshift
### QuickSight

Draw charts and explore data.

### Data Pipeline
### Glue

## Security, Identity and Compliance

Security for applications and account.

### Identity and Access Management (IAM)

Users, roles and permissions for cloud operations.

### Cloud Directory
### Cognito

Authentication and authorisation for users of your application.

### GuardDuty
### Inspector
### Macie
### Certificate Manager
### CloudHSM
### Directory Service
### Firewall Manager
### Key Management Service

Store encryption keys used to decrypt data.  (e.g. EBS or S3)  Each key has AWS Console users allowed to use it.  (giving them access to the data)   AWS has the master keys.  If AWS didn't have the key it would be better, but then it costs more to implement your own key management.

### Organisations
### Secrets Manager
### Single Sign-on
### Shield

DDOS protection for websites.

### WAF
### Artifact

## Mobile Services

Services for applications (often mobile ones)

### Mobile Hub

Manage various mobile services, similar to Google Firebase.

### Pinpoint

Analytics for mobile apps.

### AppSync

Managed GraphQL API.

### Device Farm

Test mobile apps on various real devices.

### Mobile SDK

## AR & VR

### Sumerian

Create AR & VR content.

## Application Integration

Services for connecting other services together.

### Amazon MQ


### Simple Queue Service (SQS)

Post messages to a message queue and fetch them for processing.

### Simple Notification Service (SNS)

Push notifications to services.

### Step Functions

Execute a series of lambda functions in a pipeline.

## Customer Engagement

Services to interact with customers.

### Connect

Managed call center.

### Simple Email Service

Email server in the cloud.

## Business Productivity

### Alexa

Services for organising meetings.

### Chime
### WorkDocs
### WorkMail

Mail service like Gmail.

## Desktop & App Streaming

### WorkSpaces

Desktop in the cloud.

### AppStream

## Internet of Things

### IoT Core
### FreeRTOS

An Amazon extension of FreeRTOS, an OS for embedded devices, with support added for calling AWS cloud -services.

### Greengrass
### IoT 1-click
### IoT Analytics
### IoT Button
### IoT Device Defender
### IoT Device Management

## Game Development

### GameLift
### Lumberyard

A game engine by Crytek, similar to unity/unreal.

## Software

### AWS Marketplace

Buy and sell services for AWS.

## AWS Cost Management

### Cost Explorer
### Budgets
### Reserved Instance Pricing
### Cost and Usage Report


## References

* https://www.youtube.com/watch?v=TkT4iFRkaZk









# House-Price-Prediction
Building a ‘House Price Prediction’ Project with DevOps — Focusing on Automation, IaC, CI/CD, and Monitoring


My goal is to create a fully automated pipeline, integrating infrastructure as code, CI/CD, and monitoring for continuous improvement. I will start by setting up the necessary AWS Identity and Access Management (IAM) roles to securely manage resource permissions across services, including Amazon SageMaker for training the model, AWS Lambda for model inference, and additional services like CodePipeline and CodeBuild for deployment. By the end of this project, I’ll have an end-to-end solution for predicting house prices with scalable, automated deployment capabilities.

![image](https://github.com/user-attachments/assets/7372536a-3e8c-41ea-9820-d0afa0cf2937)

### Step 1: Setting Up IAM Roles for House Price Prediction

To automate the deployment and management of our house prediction model, we’ll begin by setting up essential IAM roles. These roles will enable SageMaker and Lambda to access the necessary resources.

I will provide IAM roles specifically for SageMaker and Lambda with the appropriate permissions. This roles will allow the SageMaker to run training jobs, access S3 buckets, and manage resources needed for model training, and Lambda to log to CloudWatch and access S3 in a read-only capacity for deploying or invoking the trained model.

<b><l>1.1 Creating the SageMaker Execution Role</l></b>

In the AWS Management Console, go to IAM > Roles > Create Role.

![image](https://github.com/user-attachments/assets/0203a6dd-18b5-4de1-bd10-54113e32fda9)

![image](https://github.com/user-attachments/assets/3da1f77e-d4b7-420f-8602-0e6ccfd01b3d)

Choose AWS Service as the trusted entity and select SageMaker as the use case.

![image](https://github.com/user-attachments/assets/4970220b-879a-4f29-a851-dc8a1a91351f)

![image](https://github.com/user-attachments/assets/62e757c2-70af-41db-b9ad-8b155d818323)

Attach the following policy: AmazonSageMakerFullAccess

![image](https://github.com/user-attachments/assets/37e1420f-06b4-46d3-bfd3-1c2960c57f04)

Name the role, e.g., SageMakerExecutionRole, and create it.

![image](https://github.com/user-attachments/assets/51d38886-377b-4942-b990-1f555a76617f)


After creating the role, navigate to it, locate the ARN, and copy it for use in the CloudFormation template.

![image](https://github.com/user-attachments/assets/128c6189-65b4-4ad9-9a3e-e660f7b96483)

![image](https://github.com/user-attachments/assets/658a84c8-5918-4901-9303-9d3763f55350)

We need to add AmazonS3FullAccess permission to this role, or customize it to allow access to a specific S3 bucket. Since I haven’t created a specific bucket yet, I’ll grant full access to all S3 buckets for now. To do this, go to the Permissions tab for the role, click on the Add permissions dropdown, and search for the S3 permission. Select AmazonS3FullAccess and click Add Permission.

![image](https://github.com/user-attachments/assets/10783311-0539-4f29-946f-d15fa58c5946)

![image](https://github.com/user-attachments/assets/df518819-630d-4032-9f0c-04e35d408e05)

<l><b>1.2 Creating the Lambda Execution Role</b></l>

Repeat similar steps as above, but select Lambda as the use case. Attach the following policies:
AWSLambdaBasicExecutionRole (for CloudWatch logging)
AmazonS3ReadOnlyAccess (to access model data in S3)

Name the role, e.g., LambdaExecutionRole, and create it.

![image](https://github.com/user-attachments/assets/31c9cd40-f1f2-46a4-9bcd-86768bc883fa)

![image](https://github.com/user-attachments/assets/5d285061-b49a-4099-8e8e-c3df3e235f1a)

Once created, copy the ARN for future use.

### Step 2: Setting Up AWS Infrastructure Using CloudFormation (IaC)


Now that we’ve set up IAM roles, let’s automate the creation of the infrastructure needed for this project using AWS CloudFormation. This includes provisioning resources like an S3 bucket, SageMaker notebook instance, and an API Gateway.

<l><b>2.1 Write the CloudFormation Template</b></l>

We’ll start by creating a CloudFormation template to set up the following resources:

An S3 bucket to store datasets.
A SageMaker Notebook Instance for model training.
An API Gateway to expose the model for predictions.

Here’s the CloudFormation YAML file to create these resources, saved as house-price-prediction.yml:

```
AWSTemplateFormatVersion: '2010-09-09'
Description: Infrastructure for House Price Prediction Project

Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: house-price-prediction-bucket
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true

  SageMakerNotebook:
    Type: AWS::SageMaker::NotebookInstance
    Properties:
      NotebookInstanceName: HousePriceNotebook
      InstanceType: ml.t3.medium
      RoleArn: arn:aws:iam::your-account-id:role/SageMakerExecutionRole

  ApiGateway:
    Type: AWS::ApiGateway::RestApi
    Properties:
      Name: house-price-api
      FailOnWarnings: true

Outputs:
  S3BucketName:
    Description: S3 Bucket Name
    Value: !Ref S3Bucket
```




![image](https://github.com/user-attachments/assets/979203c5-4d8a-4eef-8055-08fdb6dacf84)# House-Price-Prediction
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


<l><b>2.2 Deploy the CloudFormation Stack</b></l>

In the AWS CloudFormation console, click Create Stack and select With new resources (standard).

![image](https://github.com/user-attachments/assets/439e0f31-1c97-4bf8-8db6-de24ecd84cac)

On Prepare template, select choose an existing template. Under Specify template, choose Upload a template file and select your house-price-prediction.yml file.

![image](https://github.com/user-attachments/assets/e0119089-0f9d-46fc-bbdc-9d7732ab15dd)

![image](https://github.com/user-attachments/assets/eb9368ac-b93f-4aa8-87ef-8d4288abd770)

Click Next, provide a stack name (e.g., HousePricePredictionInfrastructure), and proceed with default options until you reach the Review step.

![image](https://github.com/user-attachments/assets/e7580dd4-df6f-4915-9379-f3afbd825187)

Click Submit. You should see a CREATE_IN_PROGRESS status, which will change to CREATE_COMPLETE once the stack is successfully created.

However, on my first attempt, I encountered an error because the Role ARN I provided for the LambdaFunction didn’t match the correct format required by AWS.

![image](https://github.com/user-attachments/assets/914e0e90-be26-406d-aad2-a42fc099836b)

After updating the ARN for both SageMaker and Lambda to the correct format, the error was resolved.

I should see a CREATE_IN_PROGRESS status, which will change to CREATE_COMPLETE once the stack is successfully created.

![image](https://github.com/user-attachments/assets/65648e18-ffff-4a8d-b610-04c7df434774)

![image](https://github.com/user-attachments/assets/0d8c25fd-b70c-4265-95a6-50fa70fe9304)

### Step 3: Setting up and Cloning an AWS CodeCommit Repository

In the AWS Management Console, navigate to CodeCommit and create a new repository (e.g., HousePricePredictionRepo).

![image](https://github.com/user-attachments/assets/39abf6dc-d80a-4844-a2f2-61e9d3590a3b)

![image](https://github.com/user-attachments/assets/e99981d9-92e2-4b77-ad83-c0bd7aceb8a5)

Once created, I’ll see three options to clone the repository: HTTPS, SSH, and HTTPS (GRC).

![image](https://github.com/user-attachments/assets/db7f8c53-dea9-4a3e-802d-8aa6d27f5b58)

Make sure Git is installed on local machine by running:

```
git --version
```

Scroll down in the CodeCommit console to find the repository’s Clone URL. Note this URL, as you’ll use it to clone the repository locally.

![image](https://github.com/user-attachments/assets/2d164ab5-624f-4147-9d74-d59f91449fe5)

Since CodeCommit requires AWS authentication, go to the IAM section in the console.
Under Users, select your user profile. Navigate to Security Credentials and scroll down to HTTPS Git credentials for AWS CodeCommit.
Click Generate Credentials.

![image](https://github.com/user-attachments/assets/ffb7fe9b-022e-4df9-aac2-7563931c6086)

![image](https://github.com/user-attachments/assets/f0871844-7ae4-46fb-99af-988ef0a48196)

Next, we need to clone the repository locally. To do this, create a new folder on your local machine where you’d like to store the repository. Open your terminal in this folder and run the clone command using the Clone URL you noted earlier.

![image](https://github.com/user-attachments/assets/ede950de-a481-4794-9b88-d56e9b0c0043)

### Step 4: Data preparation, Model Training, and Deployment to S3 using SageMaker.

In this step, I am preparing the dataset, training a machine learning model, and saving it to S3 for later use in predictions. First, I will organize and preprocess the data, ensuring it’s clean and suitable for training. Then, I will train a house price prediction model using linear regression in SageMaker, which allows for streamlined model training on AWS. After training, I will save the model to an S3 bucket so that it can be easily accessed by our Lambda function, enabling real-time predictions when deployed via an API. This setup allows a smooth, automated workflow from data preparation to model deployment.

<l><b>4.1 Preparing Data and Training the Model with SageMaker</b></l>

Before setting up the Lambda function, I need to prepare the data and train a house price prediction model using Amazon SageMaker. Once trained, the model can be saved to S3 for later use in the Lambda function.

First, gather and organize your dataset with columns for input features (e.g., square footage, location, number of bedrooms) and the target variable (e.g., price).

I sourced the data from Google, as shown below.

![image](https://github.com/user-attachments/assets/53e48d7b-079d-4bd4-90a4-f2470467f045)


After creating or obtaining your dataset, upload it to the S3 bucket you created earlier using CloudFormation.

![image](https://github.com/user-attachments/assets/66679c45-7948-4d92-bc52-edd2b94bd90c)

![image](https://github.com/user-attachments/assets/af57cdd2-fdff-44e1-a2a2-933ee6dca16e)

Make note of the S3 URI for future reference.

Use a SageMaker notebook instance to load, preprocess, and train your model on the dataset stored in S3.

To do this, navigate to Amazon SageMaker. On the left, click on Notebook instances to locate the notebook instance created using CloudFormation.

![image](https://github.com/user-attachments/assets/0639ee1b-2911-4ca7-b2ab-e13a04386d98)

Click on Open Jupyter.

![image](https://github.com/user-attachments/assets/af867d81-1757-46ae-902a-7f6fd84e76a5)


Open the notebook instance in SageMaker and select the “conda_python3” environment to start a new Jupyter notebook.

![image](https://github.com/user-attachments/assets/c518fda2-8ebe-404f-87a9-e784ddd0ffce)

A new Jupyter notebook will open in a separate window. In the first cell, enter the following code to import the necessary libraries.

```
import boto3
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
import joblib
```

It will look like below.

![image](https://github.com/user-attachments/assets/62bcf139-7a2e-4ff1-8006-284a06f8e7c7)

Next, load the dataset you uploaded to S3 into a pandas DataFrame. Insert the code into the Jupyter notebook below the cell where you’ve imported the libraries.

```
# Define your bucket name and file path
bucket_name = 'house-price-prediction-bucket'
file_key = 'housing.csv'

# Create a session and load the dataset
s3 = boto3.client('s3')
response = s3.get_object(Bucket=bucket_name, Key=file_key)

# Read the CSV file into a pandas DataFrame
data = pd.read_csv(response['Body'])
print(data.head())  # Display the first few rows of the dataset
```

![image](https://github.com/user-attachments/assets/373e10cb-8644-4b39-9a80-be7bd4cee5a0)

The next step is to check for any missing values in the dataset. Alternatively, I can select the features I want to include in the model. For instance, I will exclude the column ocean_proximity for now.

```
# Step 1: Check for missing values
print(data.isnull().sum())

# Step 2: Feature selection (example: dropping 'ocean_proximity' for now)
features = data.drop(columns=['median_house_value', 'ocean_proximity'])
target = data['median_house_value']

# Check the processed data
print(features.head())
print(target.head())

```

After executing the code, I discovered that there are 207 missing values for the total_bedroom column.

![image](https://github.com/user-attachments/assets/12661675-6d5b-48bf-8c2d-e92e40f172a0)

There are two common strategies for handling missing values in a column. One approach is to fill the missing values with the mean, median, or mode. The second option is to remove the rows that contain missing values. I decided to fill the missing values in the column with the median.

```
# Fill missing values in 'total_bedrooms' with the median
features['total_bedrooms'].fillna(features['total_bedrooms'].median(), inplace=True)

# Check for any remaining missing values
print(features.isnull().sum())

```

![image](https://github.com/user-attachments/assets/56b9cb5a-f28f-45c4-bd44-90db9edd9030)

It should fill in the missing values and then provide a summary of any remaining missing values. Now that you have filled the missing values in the total_bedrooms column and confirmed there are no remaining missing values, you can move on to the next steps: splitting your dataset into training and testing sets, training your model, and saving the model back to S3.

First, you need to split your data into training and testing sets, which is crucial for evaluating your model’s performance on unseen data. Add the following code in a new cell

```
from sklearn.model_selection import train_test_split

# Split the data into training and testing sets (80% train, 20% test)
X_train, X_test, y_train, y_test = train_test_split(features, target, test_size=0.2, random_state=42)

print("Training set size:", X_train.shape)
print("Testing set size:", X_test.shape)
```

![image](https://github.com/user-attachments/assets/10fefd30-5b2d-4775-a4ca-96a91964b254)

Next, you can train your machine learning model. To keep things simple, you can use a linear regression model. Add the following code in another new cell:

```
from sklearn.linear_model import LinearRegression

# Initialize the model
model = LinearRegression()

# Train the model
model.fit(X_train, y_train)

# Evaluate the model
train_score = model.score(X_train, y_train)
test_score = model.score(X_test, y_test)

print("Training set score:", train_score)
print("Testing set score:", test_score)

```

But I ran into another error saying there is a missing value.
ValueError: Input X contains NaN.
Let’s solve this. Check if X_test has any missing values similar to what I did for features.

```
print(X_test.isnull().sum())
```

It appears that there are missing values in the testing data.

![image](https://github.com/user-attachments/assets/33e9add4-0c5e-4b4b-a426-663a455ce26e)

Let’s fill in the missing values in the test features.

```
X_test['total_bedrooms'].fillna(features['total_bedrooms'].median(), inplace=True)
```

After filling in the missing values, check again for any remaining NaN values:

```
# Check for any remaining missing values in both datasets
print("Training features missing values:")
print(features.isnull().sum())

print("Test features missing values:")
print(X_test.isnull().sum())
```

There still seem to be missing values in the feature set. Let’s fill those in as well.

![image](https://github.com/user-attachments/assets/1e7e119d-2092-4f90-a3ba-dd45dfb8c4dd)

```
# Fill missing values in 'total_bedrooms' for training features
features['total_bedrooms'].fillna(features['total_bedrooms'].median(), inplace=True)
```

After running the command, check if you still have missing values. It should look like below.

![image](https://github.com/user-attachments/assets/61e73c27-2d26-49b1-a28e-8fe46581106d)

Once you confirm that there are no missing values, you can train your model again:

```
# Train the model
model.fit(X_train, y_train)

# Evaluate the model
train_score = model.score(X_train, y_train)
test_score = model.score(X_test, y_test)

print("Training set score:", train_score)
print("Testing set score:", test_score)

```

![image](https://github.com/user-attachments/assets/08c25f52-4846-4474-a712-5d76dfdbc9a7)

After training the model, save it to S3 using joblib for easy loading later. Here’s how you can do that in a new cell:

```
import joblib
import os

# Define the model file path
model_file_path = 'house_price_model.joblib'

# Save the model locally
joblib.dump(model, model_file_path)

# Upload the model to S3
s3.upload_file(model_file_path, bucket_name, model_file_path)

print("Model saved to S3:", f"s3://{bucket_name}/{model_file_path}")
```

As you can see, the model has been successfully saved.

![image](https://github.com/user-attachments/assets/d530191b-9119-4ad3-a0eb-e44c2420fc94)

### Step 5: Prepare the Lambda function for Prediction.

In this step, I will develop a Lambda function that predicts house prices using the trained machine learning model. I will create the Lambda function code, package it with its dependencies, and prepare it for deployment to AWS.

<l><b>5.1 Write Lambda function code.</b></l>

I am going to create a new file named lambda_function.py inside the cloned directory in my local system. You can use any code editor you prefer (like VSCode, PyCharm, or even a simple text editor). Here’s a sample code structure for your Lambda function:

```
import json
import boto3
import joblib
import numpy as np

# Initialize S3 client
s3 = boto3.client('s3')

# Define the bucket and model file name
BUCKET_NAME = 'house-price-prediction-bucket'
MODEL_FILE_NAME = 'house_price_model.joblib'

# Load the model from S3
def load_model():
    s3.download_file(BUCKET_NAME, MODEL_FILE_NAME, '/tmp/' + MODEL_FILE_NAME)
    model = joblib.load('/tmp/' + MODEL_FILE_NAME)
    return model

# Initialize the model (load it once)
model = load_model()

def lambda_handler(event, context):
    # Parse the incoming JSON data
    input_data = json.loads(event['body'])
    
    # Convert input data to a NumPy array
    features = np.array([
        input_data['longitude'],
        input_data['latitude'],
        input_data['housing_median_age'],
        input_data['total_rooms'],
        input_data['total_bedrooms'],
        input_data['population'],
        input_data['households'],
        input_data['median_income']
    ]).reshape(1, -1)
    
    # Make prediction
    prediction = model.predict(features)
    
    # Return the prediction as a JSON response
    return {
        'statusCode': 200,
        'body': json.dumps({'predicted_house_value': prediction[0]})
    }
```

To deploy the Lambda function, you need to package the code along with any dependencies, such as joblib.

Create requirements.txt: In your local repository, create a file named requirements.txt and add joblib to it.

Open Terminal: Navigate to the location of your cloned repository in your terminal.

Check File Status: Run the following command to check the status of your files:

```
git status
```

I can see the output is showing untracked files.

![image](https://github.com/user-attachments/assets/fb47f3f0-3cd9-4116-8c1e-904c91e05306)

Use the git add command to stage the files you want to commit. This usually includes lambda_function.py and requirements.txt. I can add these files individually or stage them all at once.

```
git add .
```

![image](https://github.com/user-attachments/assets/8ded3a81-45eb-4f9f-a329-8ffb63ab4828)

Commit the staged changes with a meaningful commit message. This practice helps keep track of the modifications made in your repository.

```
git commit -m "Add Lambda function and requirements for dependencies"
```

![image](https://github.com/user-attachments/assets/d53fb7f2-3e75-4aaa-94b4-c887952acc43)

Finally, push my committed changes to the CodeCommit repository.

```
git push origin master
```

The code and files have been successfully pushed to the CodeCommit repository.

![image](https://github.com/user-attachments/assets/cebec910-e227-4910-8eb8-0afe30330e06)

### Step 6: Automating Lambda function packaging with AWS CodeBuild.

In this step, I will configure AWS CodeBuild to automate the packaging of the Lambda function. First, I will create a buildspec.yml file that defines the build process, including the installation of dependencies and the packaging of the Lambda function into a zip file. After updating the CodeCommit repository with this file, I will create a CodeBuild project, specifying the source as CodeCommit and configuring the environment settings.

<l><b>6.1 Create a Buildspec file.</b></l>

I will set up AWS CodeBuild to automate the packaging of the Lambda function.

First, you need to create a buildspec.yml file in your project directory. This file defines the build process for CodeBuild.

```
version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.8
    commands:
      - echo Installing dependencies...
      - mkdir package
      - pip install -r requirements.txt -t package/
  build:
    commands:
      - echo Packaging Lambda function...
      - cp lambda_function.py package/
      - cd package
      - zip -r ../lambda_function.zip .
artifacts:
  files:
    - lambda_function.zip
```

<l><b>6.2 Update CodeCommit Repository.</b></l>

Since we have created the buildspec.yml file in the local repository, we need to push it to the centralized location. Run the following command:

```
git add buildspec.yml
git commit -m "Add buildspec for CodeBuild"
git push origin master  # or main, depending on your branch name
```

![image](https://github.com/user-attachments/assets/ed723818-0aa0-48d1-92ec-14e4c79172d0)

Let’s check to see if it is available there.

![image](https://github.com/user-attachments/assets/1564e057-870e-44ed-9ea1-8721c52f255d)

<l><b>6.3 Create a CodeBuild Project.</b></l>

Navigate to the CodeBuild service and select Create Project. Enter a name for your project.

![image](https://github.com/user-attachments/assets/b27496d4-0671-4d01-ac0d-c58d305a41dd)

![image](https://github.com/user-attachments/assets/b52fe0c9-2b08-44d5-8f7e-a40890b45ef2)

In the source section, select CodeCommit as the source provider. Then, choose the repository you created and select the branch that contains the code you want to test.

![image](https://github.com/user-attachments/assets/6889d503-18e0-47c8-be30-3612cb4d48b4)

In the environment section, set the provision model to on-demand, choose managed image for the environment image, and select EC2 for the compute type.

![image](https://github.com/user-attachments/assets/5077b04d-8491-4afa-a458-fb4dba03a609)

Select the operating system, runtime, and image.
Next, choose a service role. This role grants CodeBuild the necessary permissions to perform its tasks.

![image](https://github.com/user-attachments/assets/363f844a-e594-488d-ab58-bc80d03f47d3)

Expand Additional configuration to view timeout settings.

Set the build timeout to define how long the build should run before timing out. Also, configure the queued timeout to specify the maximum time (in minutes) a build can remain in the queue before being automatically canceled.

![image](https://github.com/user-attachments/assets/953ce0a4-bc16-477e-83fc-51ade654ba33)

I’ll leave the remaining options in this section at their default settings.

Scroll down to the Buildspec section and select the option Use a build spec file.

![image](https://github.com/user-attachments/assets/fc93461a-c947-4e63-ba55-e9b0240ccd0d)

In the Logs section, select CloudWatch to store build logs. This ensures that logs are retained after the Docker container is removed, allowing for debugging if any errors occur.

![image](https://github.com/user-attachments/assets/1ad12c05-8c80-4454-9036-a63ecc896c75)

After that, click Create build project to complete the setup.

However, I encountered an error after clicking Create build project.

![image](https://github.com/user-attachments/assets/313eba80-87f6-420f-85b9-18afb7b9645e)

The error suggests that CodeBuild requires an IAM role to access AWS resources, similar to the roles created for SageMaker and Lambda.

To resolve this, I’ll create a new IAM role for CodeBuild. On the Attach permissions policies page, I’ll select policies that grant the necessary access:

AWSCodeCommitFullAccess: Provides access to CodeCommit repositories.
AmazonS3FullAccess: Grants access to S3 for uploads and downloads (can be customized if needed).
AWSLambda_FullAccess: Needed if the build process interacts with Lambda functions.
Alternatively, I could create a custom policy if more specific permissions are required.

![image](https://github.com/user-attachments/assets/18166a23-f4f2-4c4c-8a8f-ceea852ef5e7)

Now, go back to your CodeBuild project settings and assign the newly created IAM role under the Environment section.

As you can see, I returned to the CodeBuild settings and specified the role I created.

![image](https://github.com/user-attachments/assets/aa55c179-2c02-482c-ab61-0d84aa038510)

CodeBuild was created successfully.

![image](https://github.com/user-attachments/assets/785a2764-b701-47ac-b5f4-70c9e935785a)

Now, click Start Build to test the setup.

The build status shows success.

![image](https://github.com/user-attachments/assets/39e6d24f-8db9-4113-a1f6-01bf84470756)

With the CodeBuild build successfully completed, your packaged Lambda deployment artifact (usually a .zip file) should now be stored in the specified S3 bucket, as configured in your CodeBuild project.

However, I noticed the .zip file is missing in S3. It seems I may have overlooked a step in CodeBuild. After reviewing, I realized I missed configuring the Artifact details during the project setup. To fix this, I’ll edit the project, scroll down to the Artifacts section, select S3 as the storage location, choose the correct bucket, and specify a name for the file.

![image](https://github.com/user-attachments/assets/30ced536-060f-4adb-a3f2-997dd604e618)

Then click Update project and start the build for the updated configuration.

Now, I can see the .zip file in my S3 bucket.

![image](https://github.com/user-attachments/assets/68f54781-0104-4a31-9940-74c1b75066c5)

### Step 7: Create a CloudFormation for the Lambda function.

In this step, I’ll create a CloudFormation template that defines my Lambda function and the associated resources, such as IAM roles and S3 bucket settings.

Below is the template.

```
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      Policies:
        - PolicyName: LambdaExecutionPolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - logs:CreateLogGroup
                  - logs:CreateLogStream
                  - logs:PutLogEvents
                Resource: arn:aws:logs:*:*:*

  HousePricePredictionLambda:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: HousePricePredictionLambda
      Handler: lambda_function.lambda_handler
      Role: !GetAtt LambdaExecutionRole.Arn
      Code:
        S3Bucket: house-price-prediction-bucket
        S3Key: lambda_function.zip  
      Runtime: python3.8
      Timeout: 60
      Environment:
        Variables:
          MODEL_S3_BUCKET: house-price-prediction-bucket
          MODEL_S3_KEY: house_price_model.joblib

```

Next, create a CloudFormation stack using the YAML file just created. This stack will define my Lambda function along with the necessary IAM roles.

Navigate to the CloudFormation service. Click on Create Stack and select With new resources (standard).

![image](https://github.com/user-attachments/assets/176db79e-37fe-4c50-b906-36b67fc384fc)

In the Create Stack section, choose Upload a template file to upload the YAML file from local system.

![image](https://github.com/user-attachments/assets/0c4fc7a7-bd27-48c1-9fc6-393f28c9d731)

![image](https://github.com/user-attachments/assets/1a996bd6-f37d-4d9f-b28e-a8df71223d36)

Click Next. In the Specify stack details section, provide a Stack name.

![image](https://github.com/user-attachments/assets/8e4c5d3c-ccc6-4af7-8722-e36d7cf51307)

Review settings and click Create Stack.

![image](https://github.com/user-attachments/assets/978ea7d6-c93a-4991-a545-06327e2d11d8)

After successfully creating the stack, verify the creation of the Lambda function by navigating to the Lambda service.

![image](https://github.com/user-attachments/assets/2b8bf6ae-354c-47f5-a167-1d829d032adb)

I ran into a few errors and below is the way I resolved them.

Attempt to test Lambda function. If the test fails with an error indicating that there is no module named lambda_function, follow these troubleshooting steps:

The failure may be due to the Lambda function not loading correctly because the deployment package was compressed multiple times. Initially, the package was created by the buildspec.yml file, and selecting the .zip option caused it to be compressed again. Change this option to None to prevent double compression.

![image](https://github.com/user-attachments/assets/5d012ad5-baa8-4651-afa6-c2ce54546441)

After making this adjustment, if the Lambda function still fails, check the S3 bucket and ensure that lambda_function.zip is not stored in a nested directory. If necessary, update the S3Key in the CloudFormation template to the full path of the Lambda function:

```
S3Key: HousePricePredictionBuild/lambda_function.zip
```

If you encounter a new error indicating a missing library, such as numpy, you will need to include these libraries in the requirements.txt file that you pushed to CodeCommit. Add numpy and boto3 to this file, then push the changes again.

![image](https://github.com/user-attachments/assets/5a4917e8-b8b3-4084-8fc4-e23d975bb9a6)

### Step 8: Setting Up API Gateway

Next, I will set up an API Gateway to create a RESTful API that allows users to access our house price prediction Lambda function easily.

Navigate to the API Gateway Console and select the API that you created using the CloudFormation template.

![image](https://github.com/user-attachments/assets/4f61832b-2444-4a6e-9362-6408f1daca35)

Click on “Create Resource”

![image](https://github.com/user-attachments/assets/355807c5-f0fd-40dc-92fa-bec85138c738)

Enter a name (e.g., predict) and click “Create Resource”.

![image](https://github.com/user-attachments/assets/332190cd-6c24-414a-8506-7e2beac4d26d)

Select the newly created resource, and select “Create Method”.

![image](https://github.com/user-attachments/assets/66bae7a5-c33b-4c27-96fe-b69829444bc9)

Choose POST and select “Lambda Function” as the integration type.

![image](https://github.com/user-attachments/assets/03c7b55e-bf38-461f-bc5c-669ca5a1958d)

Check “Use Lambda Proxy integration”.
Enter the name of the Lambda Function.

![image](https://github.com/user-attachments/assets/804fa90d-f946-497e-b55f-33e152f1ec6a)

Click on the created resource and select Deploy API.

![image](https://github.com/user-attachments/assets/896dbe5e-a860-429a-a72b-bd5ef542119d)

Create a new stage (e.g., dev) and click “Deploy”.

![image](https://github.com/user-attachments/assets/e24da88c-bf3f-4f34-a91f-60a82b96b234)

Note the invoke URL provided; this will be used to call your API.

![image](https://github.com/user-attachments/assets/f970696c-dd80-45ee-956f-d8a1d89353b5)

### Step 9: Setting up CodePipeline for Lambda deployment

Set up an AWS CodePipeline to automate the deployment of Lambda function whenever I push updates to my CodeCommit repository.

Navigate to the CodePipeline console and click on Create pipeline.

![image](https://github.com/user-attachments/assets/8b2ead4d-60bc-4bc7-814a-f668f02a08db)

I am choosing Build custom pipeline.

![image](https://github.com/user-attachments/assets/4d35381e-c370-420d-9504-eedc2563b18f)

Provide a pipeline name. I am providing the name HousePricePredictionPipeline

And for Execution mode, I have chosen Superseded. Because this option will deploy only the latest code without unnecessary delays or multiple deployments for each change.

![image](https://github.com/user-attachments/assets/faa98dde-e773-4640-b502-e9d489457307)

I am choosing Existing service role as I have created a service role for this.

![image](https://github.com/user-attachments/assets/8138b94a-513e-4ef4-99c8-c1de59562c32)

I chose the Artifact location.

![image](https://github.com/user-attachments/assets/9c9339b1-a80f-45b5-a77a-32b178d9695f)

Choose the Source provider. I have stored my code in CodeCommit, so I would choose the Source provider as AWS CodeCommit.

![image](https://github.com/user-attachments/assets/702667f5-9c57-407b-bca3-4c636caf0d25)

I chose Amazon CloudWatch events for change detection. Because it provides real-time responsiveness, so you don’t have to wait for periodic checks, which can be slower.

![image](https://github.com/user-attachments/assets/5078012a-1432-4a6b-b7de-7ef023d81464)

On the Add build section, I choose other build providers because I am using CodeBuild for this project and I have already created a build project earlier for House price prediction.

![image](https://github.com/user-attachments/assets/d027af97-3ed0-407f-bd64-d19ced0f08f7)

Rest of the option, I am providing it as default.

![image](https://github.com/user-attachments/assets/8fa74dab-7acc-455b-a2a3-6da6760b9e75)

On the Add deploy stage, select AWS CloudFormation as Deploy provider.

![image](https://github.com/user-attachments/assets/478c223f-1bd6-446c-b90e-acab366d36e7)

Provide an S3 path where the file is stored in the Deploy Stage of CodePipeline.

![image](https://github.com/user-attachments/assets/0c2666a4-3a97-4c08-81b4-2a455511a4f4)

This step is just telling CloudFormation whether it is allowed to create IAM roles and policies during the stack deployment. If your template involves IAM roles (for Lambda execution or other resources), I need to grant this permission by selecting CAPABILITY_NAMED_IAM.

This is the IAM role that AWS CloudFormation will assume to perform the deployment. If don’t specify it, CodePipeline will use a default role that it creates for the deployment. I have already created one so I am selecting it here.

![Uploading image.png…]()




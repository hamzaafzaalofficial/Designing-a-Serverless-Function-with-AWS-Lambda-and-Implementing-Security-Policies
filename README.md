# Designing-a-Serverless-Function-with-AWS-Lambda-and-Implementing-Security-Policies
---
## Objectives:

1. Create a basic serverless function using AWS Lambda.
2. Implement security policies for the Lambda function.
3. Test the serverless function under different security scenarios.

## Prerequisites: 
1. Aws cli installed & Configured
2. Zip installed

## Tasks: 

## Step 1: Create a Basic AWS Lambda Function
```bash
vi myfirstfunction.py
# my-firstfunction.py
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Hello, world!'
    }


zip myfirstfunction.zip myfirstfunction.py
aws lambda create-function --function-name myfunction --runtime python3.9 --role arn:aws:iam::780621779903:role/MyLambdaRole --handler myfirstfunction.lambda_handler --zip-file fileb://myfirstfunction.zip
aws lambda invoke --function-name myfunction --payload '{}' output.txt
cat output.txt
```
---
## Step 2: Implement Security Policy 1: 
```bash

vi custom-lambda-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::alpha-bucket-my/*"
        },
        {
            "Effect": "Deny",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::beta-bucket-my/*"
        },
        {
            "Effect": "Deny",
            "Action": "s3:*",
            "Resource": "*"
        }
    ]
}


aws iam create-policy --policy-name custom-lambda-policy --policy-document file://custom-lambda-policy.json
aws iam attach-role-policy --role-name MyLambdaRole --policy-arn arn:aws:iam::7805562059903:policy/custom-lambda-policy
```

---
## Step 3: Test the Lambda Function Under Different Security Scenarios Case1: 
#creating another function for checking the above policies

```bash
vi bucketcheck.py

import boto3

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    try:
        # Attempt to list objects in the bucket
        response = s3.list_objects_v2(Bucket='beta-bucket-my')
        return {
            'statusCode': 200,
            'body': 'Access Granted',
            'objects': response.get('Contents', [])
        }
    except Exception as e:
        # Log the exception for debugging
        print(f"Error: {e}")
        return {
            'statusCode': 403,
            'body': 'Access Denied'
        }



zip bucketcheck.zip bucketcheck.py
aws lambda create-function --function-name my-bucket-check --runtime python3.9 --role arn:aws:iam::7704441068603:role/MyLambdaRole --handler bucketcheck.lambda_handler --zip-file fileb://bucketcheck.zip
aws lambda invoke --function-name my-bucket-check --payload '{}' output.txt
```

--- 

## Step 2: Implement Security Policy 2 and Testing the Lambda Function Under Different Security Scenario Case2: 

```bash

vi basicServerlessFunction.py

import os
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': f"API_KEY: {os.environ['API_KEY']}, DB_PASSWORD: {os.environ['DB_PASSWORD']}"
    }



zip basicServerlessFunction.zip basicServerlessFunction.py
aws lambda create-function --function-name basicServerlessFunction --runtime python3.9 --role arn:aws:iam::696921109903:role/MyLambdaRole --handler basicServerlessFunction.lambda_handler --zip-file fileb://basicServerlessFunction.zip

aws kms create-key --description "Lambda Env Encryption"

aws kms encrypt --key-id 4a25e2b3-b3db-4026-b167-7ca7e09c9f1d --plaintext $(echo -n "secretpass123" | base64) --query CiphertextBlob --output text
aws kms encrypt --key-id 4a25e2b3-b3db-4026-b167-7ca7e09c9f1d --plaintext $(echo -n "apikey123" | base64) --query CiphertextBlob --output text

aws lambda update-function-configuration --function-name basicServerlessFunction --environment Variables="{DB_PASSWORD=AQICAHh1KbrbmI05nhAM24vU2ZpWf4L+lfaD8pxwrQLEOUjLrgG3G5OFqMQw7FSp9Eg+mNJiAAAAazBpBgkqhkiG9w0BBwagXDBaAgEAMFUGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMio39HekZ8heAzkBXAgEQgCg5fxnX/HqRTNv1lHdw2W3rSItqxOfGd46ANjv72NGTOh49K5brtlHL,API_KEY=AQICAHh1KbrbmI05nhAM24vU2ZpWf4L+lfaD8pxwrQLEOUjLrgGoN4wsW9NRiZ47CCk+6nWrAAAAZzBlBgkqhkiG9w0BBwagWDBWAgEAMFEGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQM7a1XwAxYsYdj7SedAgEQgCRysk+sEcSe+YSvdYGotNLUwIzCMjWMaIKUVqvwjaoUOeVKe+8=}"

aws lambda invoke --function-name basicServerlessFunction --payload '{}' output.json
cat output.json
```
--- 
## Summary:

This guide successfully creates a basic serverless function with AWS Lambda, implements security measures that emulate AppArmor policies through IAM roles and environment configurations, and testes the function under different security scenarios. This approach helps to ensure that your serverless functions are secure and compliant with your
organization's security policies.

--- 






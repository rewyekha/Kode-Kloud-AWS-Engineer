# Page 1

The Nautilus DevOps team needs to implement a Lambda function using a CloudFormation stack. Create a CloudFormation template named `/root/nautilus-lambda.yml` on the AWS client host and configure it to create the following components. The stack name must be `nautilus-lambda-app`.

1. Create a Lambda function named `nautilus-lambda`.
2. Use the Runtime `Python`.
3. The function should print the body `Welcome to KKE AWS Labs!`.
4. Ensure the status code is `200`.
5. Create and use the IAM role named `lambda_execution_role`.

\
`Notes:`

* Create the resources only in `us-east-1` region.





***

\# Day 48: Automating Infrastructure Deployment with AWS CloudFormation\
\## Lab Status✅ **PASSED**\
\---\
\## Overview\
This lab demonstrates how to automate AWS infrastructure deployment using **AWS CloudFormation** and the **AWS CLI**. The objective is to define and deploy an AWS Lambda function and its required IAM execution role using infrastructure-as-code principles.\
All resources are provisioned in the **us-east-1 (N. Virginia)** AWS region.\
\---\
\## Objectives\
By completing this lab, you will be able to:\
\- Write a CloudFormation template in YAML- Create an IAM execution role for AWS Lambda- Deploy an AWS Lambda function using CloudFormation- Validate infrastructure deployment using AWS CLI- Invoke and verify Lambda execution from the CLI\
\---\
\## Prerequisites\
\- Access to an AWS account- AWS CLI installed on the `aws-client` host- Valid AWS credentials configured- Permission to create:  - CloudFormation stacks  - IAM roles  - Lambda functions\
\---\
\## AWS Credentials (Lab-Provided)\
\- **Region:** us-east-1 - **Authentication:** Provided via `showcreds` command on aws-client host \
(Do not hardcode credentials in templates or documentation.)\
\---\
\## Architecture Overview\
The CloudFormation stack provisions the following resources:\
\### IAM Role- **Name:** `lambda_execution_role`- **Purpose:** Allows Lambda to write logs to CloudWatch\
\### Lambda Function- **Name:** `nautilus-lambda`- **Runtime:** Python 3.9- **Handler:** `index.lambda_handler`- **Timeout:** 10 seconds- **Response:**  - Status Code: `200`  - Body: `Welcome to KKE AWS Labs!`\
\---\
\## CloudFormation Template\
**File Path:**\
\`\`

/root/nautilus-lambda.yml

````

### Template Content

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Nautilus Lambda Application Stack

Resources:

  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: lambda_execution_role
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

  NautilusLambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: nautilus-lambda
      Runtime: python3.9
      Handler: index.lambda_handler
      Role: !GetAtt LambdaExecutionRole.Arn
      Timeout: 10
      Code:
        ZipFile: |
          def lambda_handler(event, context):
              return {
                  "statusCode": 200,
                  "body": "Welcome to KKE AWS Labs!"
              }
````

***

### Deployment Steps (AWS CLI)

#### Step 1: Create the CloudFormation Stack

aws cloudformation create-stack \</span>  --stack-name nautilus-lambda-app \</span>  --template-body file:///root/nautilus-lambda.yml \</span>  --capabilities CAPABILITY\_NAMED\_IAM \</span>  --region us-east-1

***

#### Step 2: Wait for Stack Creation to Complete

aws cloudformation wait stack-create-complete \</span>  --stack-name nautilus-lambda-app \</span>  --region us-east-1

> ✅ No output indicates successful completion.

***

#### Step 3: Verify Stack Status

aws cloudformation describe-stacks \</span>  --stack-name nautilus-lambda-app \</span>  --region us-east-1

Expected output:

```
StackStatus: CREATE_COMPLETE
```

***

### Lambda Function Validation

#### Invoke Lambda Using CLI

aws lambda invoke \</span>  --function-name nautilus-lambda \</span>  response.json \</span>  --region us-east-1

#### View Response

cat response.json

#### Expected Result

{  "statusCode": 200,  "body": "Welcome to KKE AWS Labs!"}

***

### Verification Summary

| Requirement                  | Status |
| ---------------------------- | ------ |
| CloudFormation Stack Created | ✅      |
| Stack Name Correct           | ✅      |
| IAM Role Created             | ✅      |
| Lambda Function Created      | ✅      |
| Python Runtime               | ✅      |
| Status Code 200              | ✅      |
| Correct Response Body        | ✅      |
| Region us-east-1             | ✅      |

***

### Cleanup (Optional)

To delete all resources created by the stack:

aws cloudformation delete-stack \</span>  --stack-name nautilus-lambda-app \</span>  --region us-east-1

***

### Key Takeaways

* CloudFormation enables repeatable, consistent infrastructure deployment
* IAM permissions must be explicitly acknowledged using `CAPABILITY_NAMED_IAM`
* Lambda functions can be fully deployed without ZIP files using inline code
* AWS CLI is a powerful alternative to the AWS Console for automation

***

### Conclusion

This lab successfully demonstrates how to deploy AWS infrastructure using CloudFormation and validate it using the AWS CLI. The Lambda function executed as expected, returning the correct HTTP status and response body.



<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

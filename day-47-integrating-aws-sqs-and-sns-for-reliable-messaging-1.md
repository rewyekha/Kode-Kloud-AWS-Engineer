# Day 47: Integrating AWS SQS and SNS for Reliable Messaging

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

The Nautilus DevOps team needs to implement a Lambda function using a CloudFormation stack. Create a CloudFormation template named `/root/nautilus-lambda.yml` on the AWS client host and configure it to create the following components. The stack name must be `nautilus-lambda-app`.

1. Create a Lambda function named `nautilus-lambda`.
2. Use the Runtime `Python`.
3. The function should print the body `Welcome to KKE AWS Labs!`.
4. Ensure the status code is `200`.
5. Create and use the IAM role named `lambda_execution_role`.

`Notes:`

* Create the resources only in `us-east-1` region.

## Day 48: Automating Infrastructure Deployment with AWS CloudFormation

---

### Overview

This lab demonstrates how to automate AWS infrastructure deployment using AWS CloudFormation and the AWS CLI. The objective is to define and deploy an AWS Lambda function and its required IAM execution role using infrastructure-as-code principles.

All resources are provisioned in the **us-east-1 (N. Virginia)** AWS region.

---

### Objectives

By completing this lab, you will be able to:

* Write a CloudFormation template in YAML
* Create an IAM execution role for AWS Lambda
* Deploy an AWS Lambda function using CloudFormation
* Validate infrastructure deployment using AWS CLI
* Invoke and verify Lambda execution from the CLI

---

### Prerequisites

* Access to an AWS account
* AWS CLI installed on the `aws-client` host
* Valid AWS credentials configured
* Permission to create:
  * CloudFormation stacks
  * IAM roles
  * Lambda functions

---

### AWS Credentials

Credentials are provided via the `showcreds` command on the `aws-client` host.

| Parameter | Value           |
| --------- | --------------- |
| Region    | us-east-1       |
| Auth      | Via `showcreds` |

>  Do not hardcode credentials in templates or documentation.

---

### Architecture Overview

The CloudFormation stack provisions the following two resources:

#### IAM Role

| Property       | Value                                     |
| -------------- | ----------------------------------------- |
| Name           | `lambda_execution_role`                   |
| Purpose        | Allows Lambda to write logs to CloudWatch |
| Managed Policy | `AWSLambdaBasicExecutionRole`             |

#### Lambda Function

| Property        | Value                      |
| --------------- | -------------------------- |
| Name            | `nautilus-lambda`          |
| Runtime         | Python 3.9                 |
| Handler         | `index.lambda_handler`     |
| Timeout         | 10 seconds                 |
| Response Status | 200                        |
| Response Body   | `Welcome to KKE AWS Labs!` |

---

### CloudFormation Template

**File path:** `/root/nautilus-lambda.yml`

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
```

---

### Deployment Steps

#### Step 1 — Create the CloudFormation Stack

```bash
aws cloudformation create-stack \
  --stack-name nautilus-lambda-app \
  --template-body file:///root/nautilus-lambda.yml \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1
```

> `CAPABILITY_NAMED_IAM` is required whenever your template creates IAM resources with custom names.

---

#### Step 2 — Wait for Stack Creation to Complete

```bash
aws cloudformation wait stack-create-complete \
  --stack-name nautilus-lambda-app \
  --region us-east-1
```

>  No output indicates successful completion. The command blocks until the stack reaches `CREATE_COMPLETE` or fails.

---

#### Step 3 — Verify Stack Status

```bash
aws cloudformation describe-stacks \
  --stack-name nautilus-lambda-app \
  --region us-east-1
```

**Expected output (partial):**

```json
{
    "Stacks": [
        {
            "StackName": "nautilus-lambda-app",
            "StackStatus": "CREATE_COMPLETE"
        }
    ]
}
```

---

### Lambda Function Validation

#### Invoke the Lambda Function

```bash
aws lambda invoke \
  --function-name nautilus-lambda \
  response.json \
  --region us-east-1
```

#### View the Response

```bash
cat response.json
```

**Expected result:**

```json
{
    "statusCode": 200,
    "body": "Welcome to KKE AWS Labs!"
}
```

---

### Verification Summary

| Check                           | Expected Result                | Status |
| ------------------------------- | ------------------------------ | ------ |
| Stack status                    | `CREATE_COMPLETE`              |       |
| IAM role created                | `lambda_execution_role` exists |       |
| Lambda function created         | `nautilus-lambda` exists       |       |
| Lambda invocation response code | `200`                          |       |
| Lambda response body            | `Welcome to KKE AWS Labs!`     |       |

---

### Cleanup (Optional)

To delete all resources created by the stack:

```bash
aws cloudformation delete-stack \
  --stack-name nautilus-lambda-app \
  --region us-east-1
```

> This will remove the Lambda function and the IAM role. Confirm deletion by checking the CloudFormation console or running `describe-stacks` again.

---

### Key Takeaways

* **CloudFormation** enables repeatable, consistent infrastructure deployment through code
* **`CAPABILITY_NAMED_IAM`** must be explicitly passed when templates create named IAM resources
* Lambda functions can be fully deployed **without ZIP files** using the `ZipFile` inline code block
* The **AWS CLI** is a powerful alternative to the AWS Console for automation and scripting

---

### Conclusion

This lab successfully demonstrates how to deploy AWS infrastructure using CloudFormation and validate it using the AWS CLI. The Lambda function executed as expected, returning the correct HTTP status code and response body.

---

<figure><img src=".gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

---

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

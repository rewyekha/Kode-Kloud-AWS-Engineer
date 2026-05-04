# Day 34: Create a Lambda Function Using CLI

The Nautilus DevOps team continues to explore serverless architecture by setting up another Lambda function. This time, the task must be completed using the AWS Console to familiarize the team with the web interface. The function will return a custom greeting and demonstrate the capabilities of AWS Lambda effectively.

1. Create Python Script: Create a Python script named `lambda_function.py` with a function that returns the body `Welcome to KKE AWS Labs!` and status code `200`.
2. Zip the Python Script: Zip the script into a file named `function.zip`.
3. Create Lambda Function: Create a Lambda function named `xfusion-lambda-cli` using the zipped file and specify `Python` as the runtime.
4. IAM Role: Use the IAM role named `lambda_execution_role`. Use AWS CLI which is already configured on the `aws-client` host.

## AWS Lambda: Create and Deploy a Python Serverless Function via CLI

> **Platform:** KodeKloud | **Cloud:** AWS | **Region:** us-east-1 **Difficulty:** Beginner | **Topic:** AWS Lambda, Python, IAM, Serverless, AWS CLI

***

### Table of Contents

1. [Lab Overview](#lab-overview)
2. [Lab Objectives](#lab-objectives)
3. [Prerequisites](#prerequisites)
4. [Architecture](#architecture)
5. [Phase 1: Set Environment Variables](#phase-1-set-environment-variables)
6. [Phase 2: Create the Python Script](#phase-2-create-the-python-script)
7. [Phase 3: Zip the Script](#phase-3-zip-the-script)
8. [Phase 4: Get the IAM Role ARN](#phase-4-get-the-iam-role-arn)
9. [Phase 5: Create the Lambda Function](#phase-5-create-the-lambda-function)
10. [Phase 6: Invoke and Verify the Function](#phase-6-invoke-and-verify-the-function)
11. [Lab Complete](#lab-complete)
12. [Key Concepts](#key-concepts)
13. [Resource Reference](#resource-reference)

***

### Lab Overview

The Nautilus DevOps team continues to explore serverless architecture by setting up a Lambda function using the AWS CLI. The function returns a custom greeting and demonstrates the core capabilities of AWS Lambda — event-driven, stateless execution without server management.

***

### Lab Objectives

1. Create a Python script named `lambda_function.py` that returns body `Welcome to KKE AWS Labs!` with status code `200`.
2. Zip the script into a file named `function.zip`.
3. Create a Lambda function named `xfusion-lambda-cli` using the zipped file with Python as the runtime.
4. Use the existing IAM role named `lambda_execution_role`.

***

### Prerequisites

| Field                | Value                            |
| -------------------- | -------------------------------- |
| Region               | `us-east-1`                      |
| Access Method        | AWS CLI on `aws-client` host     |
| Lambda Function Name | `xfusion-lambda-cli`             |
| Runtime              | `python3.9`                      |
| IAM Role             | `lambda_execution_role`          |
| Handler              | `lambda_function.lambda_handler` |

***

### Architecture

```bash
aws-client host
      │
      │ 1. Write lambda_function.py
      │ 2. zip → function.zip
      │ 3. aws lambda create-function
      │
      ▼
┌─────────────────────────────────────────────────────┐
│                  AWS us-east-1                      │
│                                                     │
│   ┌──────────────────────────────────────────────┐  │
│   │         xfusion-lambda-cli                   │  │
│   │         Runtime: python3.9                   │  │
│   │         Handler: lambda_function.lambda_handler│ │
│   │         Memory: 128MB | Timeout: 3s           │  │
│   │         Architecture: x86_64                  │  │
│   │                                               │  │
│   │   IAM Role: lambda_execution_role             │  │
│   │   Log Group: /aws/lambda/xfusion-lambda-cli   │  │
│   └──────────────────────────────────────────────┘   │
│                        │                             │
│                        │ invoke                      │
│                        ▼                             │
│              {"statusCode": 200,                     │
│               "body": "Welcome to KKE AWS Labs!"}    │
└─────────────────────────────────────────────────────┘
```

***

### Phase 1: Set Environment Variables

Define all configuration values as shell variables for reuse across commands:

```bash
LAMBDA_NAME="xfusion-lambda-cli"
RUNTIME="python3.9"
IAM_ROLE="lambda_execution_role"
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  LAMBDA_NAME="xfusion-lambda-cli"
RUNTIME="python3.9"
IAM_ROLE="lambda_execution_role"

~ on ☁️  (us-east-1) ➜
```

***

### Phase 2: Create the Python Script

Write the Lambda handler function to `lambda_function.py` using a heredoc. The function returns a dictionary with HTTP status code `200` and the required body text:

```bash
cat > lambda_function.py <<EOF
def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": "Welcome to KKE AWS Labs!"
    }
EOF
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  cat > lambda_function.py <<EOF
def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": "Welcome to KKE AWS Labs!"
    }
EOF

~ on ☁️  (us-east-1) ➜
```

The resulting `lambda_function.py` file contains:

```python
def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": "Welcome to KKE AWS Labs!"
    }
```

The function name `lambda_handler` and file name `lambda_function.py` together form the handler reference `lambda_function.lambda_handler` — the format AWS Lambda uses to locate and invoke the function.

***

### Phase 3: Zip the Script

Package the Python script into a deployment ZIP archive. AWS Lambda requires function code to be uploaded as a ZIP file:

```bash
zip function.zip lambda_function.py
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  zip function.zip lambda_function.py
  adding: lambda_function.py (deflated 14%)

~ on ☁️  (us-east-1) ➜
```

The file was compressed with `deflated 14%` reduction. The resulting `function.zip` contains `lambda_function.py` at the root level — the placement inside the ZIP determines the handler path.

***

### Phase 4: Get the IAM Role ARN

Retrieve the full Amazon Resource Name (ARN) of the existing `lambda_execution_role` IAM role. Lambda requires a role ARN — not just a role name — when creating a function:

```bash
ROLE_ARN=$(aws iam get-role \
  --role-name $IAM_ROLE \
  --query "Role.Arn" \
  --output text)
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  ROLE_ARN=$(aws iam get-role \
  --role-name $IAM_ROLE \
  --query "Role.Arn" \
  --output text)

~ on ☁️  (us-east-1) ➜
```

The ARN was stored in `$ROLE_ARN` with the format:

```
arn:aws:iam::495779504297:role/lambda_execution_role
```

***

### Phase 5: Create the Lambda Function

Deploy the Lambda function using the zipped code, IAM role, runtime, and handler configuration:

```bash
aws lambda create-function \
  --function-name $LAMBDA_NAME \
  --runtime $RUNTIME \
  --role "$ROLE_ARN" \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
```

**Terminal Output:**

```bash
~ on ☁️  (us-east-1) ➜  aws lambda create-function \
  --function-name $LAMBDA_NAME \
  --runtime $RUNTIME \
  --role "$ROLE_ARN" \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
{
    "FunctionName": "xfusion-lambda-cli",
    "FunctionArn": "arn:aws:lambda:us-east-1:495779504297:function:xfusion-lambda-cli",
    "Runtime": "python3.9",
    "Role": "arn:aws:iam::495779504297:role/lambda_execution_role",
    "Handler": "lambda_function.lambda_handler",
    "CodeSize": 293,
    "Description": "",
    "Timeout": 3,
    "MemorySize": 128,
    "LastModified": "2026-03-28T05:58:43.660+0000",
    "CodeSha256": "f2PrOe1y8nr4HY76L7eBw9MklHiu+Ha60wLK2h9i5dE=",
    "Version": "$LATEST",
    "TracingConfig": {
        "Mode": "PassThrough"
    },
    "RevisionId": "52d88c1a-cabc-4b67-b9fe-ed548327a1e1",
    "State": "Pending",
    "StateReason": "The function is being created.",
    "StateReasonCode": "Creating",
    "PackageType": "Zip",
    "Architectures": [
        "x86_64"
    ],
    "EphemeralStorage": {
        "Size": 512
    },
    "SnapStart": {
        "ApplyOn": "None",
        "OptimizationStatus": "Off"
    },
    "RuntimeVersionConfig": {
        "RuntimeVersionArn": "arn:aws:lambda:us-east-1::runtime:e753a51552fe67962908c0ac2460f73995d66390004e2d300f2697685ca61d92"
    },
    "LoggingConfig": {
        "LogFormat": "Text",
        "LogGroup": "/aws/lambda/xfusion-lambda-cli"
    }
}
```

The function was accepted by AWS with the following key configuration confirmed in the response:

| Field          | Value                                                               |
| -------------- | ------------------------------------------------------------------- |
| `FunctionName` | `xfusion-lambda-cli`                                                |
| `FunctionArn`  | `arn:aws:lambda:us-east-1:495779504297:function:xfusion-lambda-cli` |
| `Runtime`      | `python3.9`                                                         |
| `Role`         | `arn:aws:iam::495779504297:role/lambda_execution_role`              |
| `Handler`      | `lambda_function.lambda_handler`                                    |
| `CodeSize`     | `293 bytes`                                                         |
| `Timeout`      | `3 seconds`                                                         |
| `MemorySize`   | `128 MB`                                                            |
| `State`        | `Pending` (being created)                                           |
| `PackageType`  | `Zip`                                                               |
| `Architecture` | `x86_64`                                                            |
| `LogGroup`     | `/aws/lambda/xfusion-lambda-cli`                                    |

***

### Phase 6: Invoke and Verify the Function

Test the deployed Lambda function by invoking it and capturing the output to `output.json`:

```bash
aws lambda invoke \
  --function-name $LAMBDA_NAME \
  output.json
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  aws lambda invoke \
  --function-name $LAMBDA_NAME \
  output.json
{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}
```

The invocation returned `StatusCode: 200` and executed the `$LATEST` version. Read the function response from `output.json`:

```bash
cat output.json
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  cat output.json
{"statusCode": 200, "body": "Welcome to KKE AWS Labs!"}

~ on ☁️  (us-east-1) ➜
```

The function returned exactly the expected payload:

* `statusCode: 200`
* `body: Welcome to KKE AWS Labs!`

***

### Lab Complete

| Requirement       | Detail                                                              | Status    |
| ----------------- | ------------------------------------------------------------------- | --------- |
| Python script     | `lambda_function.py` with `statusCode: 200` and correct body        | Confirmed |
| ZIP archive       | `function.zip` containing `lambda_function.py`                      | Confirmed |
| Function name     | `xfusion-lambda-cli`                                                | Confirmed |
| Runtime           | `python3.9`                                                         | Confirmed |
| IAM Role          | `lambda_execution_role`                                             | Confirmed |
| Handler           | `lambda_function.lambda_handler`                                    | Confirmed |
| Function ARN      | `arn:aws:lambda:us-east-1:495779504297:function:xfusion-lambda-cli` | Confirmed |
| Invocation status | `StatusCode: 200`                                                   | Confirmed |
| Response body     | `Welcome to KKE AWS Labs!`                                          | Confirmed |

***

### Key Concepts

#### Lambda Handler Naming Convention

The handler reference `lambda_function.lambda_handler` follows the pattern `<filename>.<function_name>`:

```
lambda_function.lambda_handler
      │                │
      │                └── Function name inside the Python file
      └── Python filename without .py extension
```

AWS Lambda uses this reference to locate and invoke the correct function inside the ZIP archive. If the file were named `main.py` with a function called `handler`, the handler reference would be `main.handler`.

#### `event` and `context` Parameters

Every Lambda handler receives two parameters:

| Parameter | Type   | Contains                                                                   |
| --------- | ------ | -------------------------------------------------------------------------- |
| `event`   | dict   | Input data passed to the function (from API Gateway, S3, SNS, etc.)        |
| `context` | object | Runtime metadata — function name, memory limit, request ID, remaining time |

In this lab, both parameters are received but not used — the function simply returns a static response regardless of input.

#### `fileb://` vs `file://` Prefix

When uploading binary files to AWS CLI, the prefix matters:

| Prefix     | Use Case                                  |
| ---------- | ----------------------------------------- |
| `file://`  | Plain text files                          |
| `fileb://` | Binary files (ZIP archives, images, etc.) |

`fileb://function.zip` is required because ZIP files are binary. Using `file://` would cause the CLI to interpret the file as text and corrupt the upload.

#### Lambda Function States

When the function was first created, it returned `State: Pending`. Lambda functions progress through states:

```
Pending → Active (ready to invoke)
```

The `Pending` state means AWS is setting up the execution environment. Invoking the function while in `Pending` may queue the request until it transitions to `Active`. In this lab, the invocation succeeded because the state transitioned quickly.

#### IAM Role for Lambda

The `lambda_execution_role` grants Lambda the permissions it needs to run. At minimum, a Lambda execution role requires:

```json
{
  "Effect": "Allow",
  "Action": [
    "logs:CreateLogGroup",
    "logs:CreateLogStream",
    "logs:PutLogEvents"
  ],
  "Resource": "arn:aws:logs:*:*:*"
}
```

This allows Lambda to write execution logs to CloudWatch — which is confirmed in the response by the `LogGroup: /aws/lambda/xfusion-lambda-cli` field. Without this role, the function would fail to initialise.

***

### Resource Reference

| Resource      | Type       | Value                                                               |
| ------------- | ---------- | ------------------------------------------------------------------- |
| Function Name | Lambda     | `xfusion-lambda-cli`                                                |
| Function ARN  | AWS ARN    | `arn:aws:lambda:us-east-1:495779504297:function:xfusion-lambda-cli` |
| Runtime       | Lambda     | `python3.9`                                                         |
| Handler       | Lambda     | `lambda_function.lambda_handler`                                    |
| IAM Role      | IAM        | `lambda_execution_role`                                             |
| Role ARN      | AWS ARN    | `arn:aws:iam::495779504297:role/lambda_execution_role`              |
| Code Size     | Bytes      | `293`                                                               |
| Memory        | Lambda     | `128 MB`                                                            |
| Timeout       | Lambda     | `3 seconds`                                                         |
| Architecture  | Compute    | `x86_64`                                                            |
| Package Type  | Lambda     | `Zip`                                                               |
| Log Group     | CloudWatch | `/aws/lambda/xfusion-lambda-cli`                                    |
| Code SHA256   | Hash       | `f2PrOe1y8nr4HY76L7eBw9MklHiu+Ha60wLK2h9i5dE=`                      |
| Revision ID   | Lambda     | `52d88c1a-cabc-4b67-b9fe-ed548327a1e1`                              |
| Region        | AWS        | `us-east-1`                                                         |
| Account ID    | AWS        | `495779504297`                                                      |

***

_Lab completed on 2026-03-28 | AWS Region: us-east-1 | Runtime: python3.9 | Platform: KodeKloud_

<figure><img src=".gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

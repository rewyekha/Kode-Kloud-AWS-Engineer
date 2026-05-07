# Day 20: Create IAM Role for EC2 with Policy Attachment

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

When establishing infrastructure on the AWS cloud, Identity and Access Management (IAM) is among the first and most critical services to configure. IAM facilitates the creation and management of user accounts, groups, roles, policies, and other access controls. The Nautilus DevOps team is currently in the process of configuring these resources and has outlined the following requirements:

Create an IAM role as below:

1\) IAM role name must be `iamrole_james`.

2\) Entity type must be `AWS Service` and use case must be `EC2`.

3\) Attach a policy named `iampolicy_james`.

Use the below given AWS Credentials: (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

| Console URL | https://<aws-account>.signin.aws.amazon.com/console?region=us-east-1 |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Username    | <REDACTED_USERNAME> |
| Password    | <REDACTED_PASSWORD> |
| Start Time  | Tue Mar 03 04:20:14 UTC 2026                                                                                                               |
| End Time    | Tue Mar 03 05:20:14 UTC 2026                                                                                                               |

`Notes:`

* Create the resources only in `us-east-1` region.
* To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:\
  ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)

## Day 20 – Create IAM Role for EC2 with Policy Attachment

###  Lab Objective

Create an IAM Role in **us-east-1** with the following configuration:

| Requirement    | Value             |
| -------------- | ----------------- |
| Role Name      | `iamrole_james`   |
| Trusted Entity | AWS Service       |
| Use Case       | EC2               |
| Attach Policy  | `iampolicy_james` |
| Region         | us-east-1         |

Service Used: **AWS Identity and Access Management (IAM)**\
Cloud Provider: **Amazon Web Services (AWS)**

---

##  Lab Questions

1. How do you create an IAM Role for EC2 using AWS Console?
2. How do you create the same IAM Role using AWS CLI?
3. How do you verify the role and attached policy?
4. What common errors may occur and how do you troubleshoot them?

---

##  Solution 1: Using AWS Console (GUI Method)

---

###  Step 1: Login to AWS Console

* Open AWS Console
* Set region (top-right) to:

```bash
us-east-1 (N. Virginia)
```

 Always verify region before creating resources.

---

###  Step 2: Navigate to IAM

1. Search for **IAM**
2. Click **Roles**
3. Click **Create role**

---

###  Step 3: Select Trusted Entity

* Trusted entity type → **AWS service**
* Use case → **EC2**
* Click **Next**

This allows EC2 instances to assume this role.

---

###  Step 4: Attach Policy

Search and select:

```
iampolicy_james
```

Click **Next**

---

###  Step 5: Name the Role

Enter:

```
iamrole_james
```

Click **Create role**

---

###  Verification (GUI)

IAM → Roles → Search `iamrole_james`

Verify:

* Trusted entity = EC2
* Attached policy = iampolicy\_james

---

##  Solution 2: Using AWS CLI

---

###  Step 1: Create Trust Policy File

```bash
vi trust-policy.json
```

Paste:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Save and exit.

---

###  Step 2: Create IAM Role

```bash
aws iam create-role \
  --role-name iamrole_james \
  --assume-role-policy-document file://trust-policy.json \
  --region us-east-1
```

####  Terminal Output

```json
{
    "Role": {
        "Path": "/",
        "RoleName": "iamrole_james",
        "RoleId": "AROARFNRRJG2FN5X7NZHW",
        "Arn": "arn:aws:iam::080366094772:role/iamrole_james",
        "CreateDate": "2026-03-03T04:22:46Z",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "ec2.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        }
    }
}
```

####  What This Means

* Role successfully created
* ARN confirms correct account and role name
* Trust relationship allows EC2 service

---

###  Step 3: Attach Policy

```bash
aws iam attach-role-policy \
  --role-name iamrole_james \
  --policy-arn arn:aws:iam::080366094772:policy/iampolicy_james \
  --region us-east-1
```

(No output means success.)

---

###  Step 4: Verify Role

```bash
aws iam get-role --role-name iamrole_james --region us-east-1
```

#### Output

```json
{
    "Role": {
        "RoleName": "iamrole_james",
        "Arn": "arn:aws:iam::080366094772:role/iamrole_james",
        "MaxSessionDuration": 3600
    }
}
```

---

###  Step 5: Verify Attached Policy

```bash
aws iam list-attached-role-policies \
  --role-name iamrole_james \
  --region us-east-1
```

#### Output

```json
{
    "AttachedPolicies": [
        {
            "PolicyName": "iampolicy_james",
            "PolicyArn": "arn:aws:iam::080366094772:policy/iampolicy_james"
        }
    ]
}
```

 Confirms policy successfully attached.

---

##  Common Errors & Troubleshooting

---

###  1. MalformedPolicyDocument

```
MalformedPolicyDocument: Syntax errors in policy
```

#### Cause:

* JSON formatting issue
* Missing commas or brackets

#### Fix:

Validate JSON:

```bash
cat trust-policy.json
```

Use JSON validator if needed.

---

###  2. NoSuchEntity (Policy Not Found)

```
An error occurred (NoSuchEntity) when calling the AttachRolePolicy operation
```

#### Cause:

* Policy name incorrect
* Wrong ARN
* Policy not created

#### Fix:

List policies:

```bash
aws iam list-policies --scope Local --region us-east-1
```

---

###  3. AccessDenied

```
AccessDenied: User is not authorized to perform iam:CreateRole
```

#### Cause:

* IAM permissions missing

#### Fix:

Ensure user has IAM administrative permissions.

---

###  4. Wrong Region

Check configured region:

```bash
aws configure get region
```

If incorrect:

```bash
aws configure
```

Set to:

```
us-east-1
```

---

###  5. Expired Credentials

```
ExpiredToken: The security token included in the request is expired
```

#### Fix:

* Run `showcreds`
* Reconfigure AWS CLI
* Ensure lab time is valid

---

##  Command Summary

| Purpose       | Command                               |
| ------------- | ------------------------------------- |
| Create Role   | `aws iam create-role`                 |
| Attach Policy | `aws iam attach-role-policy`          |
| Verify Role   | `aws iam get-role`                    |
| Verify Policy | `aws iam list-attached-role-policies` |

---

##  Final Outcome

 IAM Role `iamrole_james` created\
 Trusted entity: EC2\
 Policy `iampolicy_james` attached\
 Region: us-east-1

---

<figure><img src=".gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

---

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

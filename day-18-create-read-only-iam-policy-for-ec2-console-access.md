# Day 18: Create Read-Only IAM Policy for EC2 Console Access

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

When establishing infrastructure on the AWS cloud, Identity and Access Management (IAM) is among the first and most critical services to configure. IAM facilitates the creation and management of user accounts, groups, roles, policies, and other access controls. The Nautilus DevOps team is currently in the process of configuring these resources and has outlined the following requirements.

Create an IAM policy named `iampolicy_jim` in `us-east-1` region, it must allow read-only access to the EC2 console, i.e this policy must allow users to view all instances, AMIs, and snapshots in the Amazon EC2 console.

Use below given AWS Credentials: (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

| Console URL | https://<aws-account>.signin.aws.amazon.com/console?region=us-east-1 |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Username    | <REDACTED_USERNAME> |
| Password    | <REDACTED_PASSWORD> |
| Start Time  | Sun Mar 01 11:16:11 UTC 2026                                                                                                               |
| End Time    | Sun Mar 01 12:16:11 UTC 2026                                                                                                               |

`Notes:`

* Create the resources only in `us-east-1` region.
* To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:\
  ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)

Here is your complete **GitBook documentation** covering:

*  GUI Method
*  CLI Method (with your actual output)
*  How to write IAM JSON
*  Where to get policy JSON
*  IAM Policy rules & structure

---

###  Objective

Create an IAM policy named:

```
iampolicy_jim
```

In region:

```
us-east-1
```

The policy must allow **read-only access** to the EC2 console (view instances, AMIs, snapshots, etc.).

We will use:

* **AWS Identity and Access Management**
* For granting access to **Amazon EC2**

---

##  Method 1: Using AWS Console (GUI)

### Step 1: Login to AWS Console

Login using provided credentials.

Ensure region (top-right corner) is set to:

```
N. Virginia (us-east-1)
```

---

### Step 2: Navigate to IAM

1. Search for **IAM**
2. Click **Policies**
3. Click **Create policy**

---

### Step 3: Use JSON Tab

Select **JSON** and paste:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*"
      ],
      "Resource": "*"
    }
  ]
}
```

Click **Next**

---

### Step 4: Name the Policy

Policy Name:

```
iampolicy_jim
```

Click **Create policy**

---

###  Verify in Console

Go to:

IAM → Policies → Search `iampolicy_jim`

Confirm:

* Type: Customer managed
* Permissions: ec2:Describe\*

---

##  Method 2: Using AWS CLI

---

###  Step 1: Set Region

```bash
~ on   (us-east-1)   aws configure set region us-east-1
```

---

###  Step 2: Create Policy JSON File

```bash
~ on   (us-east-1)   vi iampolicy_jim.json
```

Paste:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*"
      ],
      "Resource": "*"
    }
  ]
}
```

Save and exit.

---

###  Step 3: Create IAM Policy

```bash
~ on   (us-east-1)   aws iam create-policy \
  --policy-name iampolicy_jim \
  --policy-document file://iampolicy_jim.json \
  --region us-east-1
```

####  Your CLI Output

```json
{
    "Policy": {
        "PolicyName": "iampolicy_jim",
        "PolicyId": "ANPA3EZALATIVPTEGXZOU",
        "Arn": "arn:aws:iam::766182622417:policy/iampolicy_jim",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2026-03-01T11:20:22Z",
        "UpdateDate": "2026-03-01T11:20:22Z"
    }
}
```

---

###  Step 4: Verify Policy Exists

```bash
~ on   (us-east-1)   aws iam list-policies --scope Local --region us-east-1
```

Output includes:

```json
{
    "PolicyName": "iampolicy_jim",
    "PolicyId": "ANPA3EZALATIVPTEGXZOU",
    "Arn": "arn:aws:iam::766182622417:policy/iampolicy_jim",
    "CreateDate": "2026-03-01T11:20:22Z"
}
```

---

#### Filter Specific Policy

```bash
~ on   (us-east-1)   aws iam list-policies --scope Local \
  --query "Policies[?PolicyName=='iampolicy_jim']"
```

Output:

```json
[
    {
        "PolicyName": "iampolicy_jim",
        "PolicyId": "ANPA3EZALATIVPTEGXZOU",
        "Arn": "arn:aws:iam::766182622417:policy/iampolicy_jim",
        "CreateDate": "2026-03-01T11:20:22Z"
    }
]
```

---

##  Where Do We Get the Policy JSON?

There are **3 main ways**:

---

### 1 AWS Documentation (Recommended Source)

AWS provides official documentation listing all EC2 actions:

* EC2 API Permissions Reference
* IAM JSON Policy Reference

Search for:

```
AWS EC2 IAM actions reference
```

This helps you identify:

* `Describe*` (read-only)
* `StartInstances`
* `StopInstances`
* etc.

---

### 2 Use AWS Managed Policies as Reference

You can inspect AWS managed policy:

```
AmazonEC2ReadOnlyAccess
```

View its JSON and replicate minimal permissions.

---

### 3 Write It Yourself (Custom Policy)

To allow read-only access:

You only need:

```
ec2:Describe*
```

Because:

* All read/view operations in EC2 start with **Describe**
* Write actions include:
  * RunInstances
  * TerminateInstances
  * Create\*
  * Delete\*
  * Modify\*

So we restrict to:

```
Describe*
```

---

##  IAM Policy JSON Structure Explained

Every IAM policy contains:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [],
      "Resource": ""
    }
  ]
}
```

####  Version

Always use:

```
2012-10-17
```

####  Effect

* `"Allow"`
* or `"Deny"`

####  Action

Defines what operations are allowed.

Example:

```
ec2:Describe*
```

####  Resource

Defines which resources.

For read-only:

```
"*"
```

means all EC2 resources.

---

##  IAM Policy Rules & Best Practices

 Follow **Least Privilege Principle**\
Grant only required permissions.

 Avoid `"Action": "*"`

 Restrict resource scope when possible

 Test policy before attaching to production users

---

##  Final Result

 Policy Name: `iampolicy_jim`\
 Region: `us-east-1`\
 Access: Read-only EC2 console\
 Created via: GUI & CLI\
 Verified via CLI

---

##  Conclusion

The IAM policy **iampolicy\_jim** was successfully created using both:

* AWS Console (GUI)
* AWS CLI

It grants read-only access to EC2 resources following IAM best practices.

---

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

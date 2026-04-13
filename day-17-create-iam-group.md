# Day 17: Create IAM Group

The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.

Create an IAM group named `iamgroup_ammar`.\
<br>

<br>

Use below given AWS Credentials: (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

| Console URL | [https://544126452056.signin.aws.amazon.com/console?region=us-east-1](https://544126452056.signin.aws.amazon.com/console?region=us-east-1) |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Username    | kk\_labs\_user\_863563                                                                                                                     |
| Password    | Cm1GYtwQ!5or                                                                                                                               |
| Start Time  | Sat Feb 28 16:48:17 UTC 2026                                                                                                               |
| End Time    | Sat Feb 28 17:48:17 UTC 2026                                                                                                               |

\
`Notes:`

* Create the resources only in `us-east-1` region.
* To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:\
  ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)





## Create IAM Group: `iamgroup_ammar`

### Overview

The Nautilus DevOps team is provisioning cloud resources in **Amazon Web Services (AWS)** as part of a structured migration strategy.

This document outlines the steps to create an IAM group named:

```
iamgroup_ammar
```

using the **us-east-1** region.

***

### Service Used

This task uses **AWS Identity and Access Management (IAM)**.

IAM allows you to:

* Create users
* Create groups
* Assign permissions via policies
* Control access to AWS resources securely

An **IAM group** is a collection of IAM users that share the same permissions.

***

### Prerequisites

* AWS Console access credentials
* Access to `aws-client` machine (for CLI method)
* Region set to: `us-east-1`

***

## Method 1: Using AWS Console

#### Step 1: Login

Use the provided AWS Console URL and credentials.

#### Step 2: Navigate to IAM

1. Go to **Services**
2. Search for **IAM**
3. Open the IAM Dashboard

#### Step 3: Create the Group

1. Click **User groups**
2. Click **Create group**
3. Enter:

```
iamgroup_ammar
```

4. Skip attaching policies (unless specified)
5. Click **Create group**

***

## Method 2: Using AWS CLI (aws-client host)

#### Step 1: Create the IAM Group

```bash
aws iam create-group --group-name iamgroup_ammar --region us-east-1
```

#### Step 2: Verify the Group

```bash
aws iam list-groups --region us-east-1
```

#### Expected Output

You should see:

```json
{
    "Groups": [
        {
            "GroupName": "iamgroup_ammar",
            "Arn": "arn:aws:iam::<account-id>:group/iamgroup_ammar"
        }
    ]
}
```

***

## Verification

The task is successful if:

* The group appears in `aws iam list-groups`
* The group is visible in the IAM Console under **User groups**
* Region is confirmed as `us-east-1`

***

## Best Practices

* Use meaningful group names
* Attach policies to groups instead of individual users
* Follow the principle of least privilege
* Always verify the active AWS region before creating resources

***

## Conclusion

The IAM group `iamgroup_ammar` has been successfully created in the `us-east-1` region.

This setup enables centralized permission management for users assigned to this group.


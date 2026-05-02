# Day 19: Attach IAM Policy to IAM User

The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.

An IAM user named `iamuser_siva` and a policy named `iampolicy_siva` already exist. Attach the IAM policy `iampolicy_siva` to the IAM user `iamuser_siva`.

> **Note:** Run `showcreds` on the `aws-client` host to retrieve temporary AWS credentials, then configure the AWS CLI using `aws configure`.

**Notes:**

* Create the resources only in `us-east-1` region.
  ## Day 19: Attach IAM Policy to IAM User (AWS Task Guide)

***

### What is the objective?

Attach an existing IAM policy:

```
iampolicy_siva
```

To an existing IAM user:

```
iamuser_siva
```

Region requirement:

```
us-east-1
```

***

## Step 1: Login to AWS Console

Use the provided credentials:

* **Console URL:**
  `the AWS Management Console URL
* **Region:** us-east-1

* Ensure the region is set to **N. Virginia (us-east-1)** from the top-right corner.

***

## Method 1: Attach Policy Using AWS Console

***

### How to attach the policy?

#### 1. Go to IAM Dashboard

Search for **IAM** in AWS services.

Open:

### AWS Identity and Access Management

***

#### 2. Navigate to Users

* Click **Users**
* Select user:

```
iamuser_siva
```

***

#### 3. Add Permission

* Go to **Permissions** tab
* Click **Add permissions**
* Choose:

```
Attach policies directly
```

* Search for:

```
iampolicy_siva
```

* Select the checkbox
* Click **Next**
* Click **Add permissions**

***

### Verification

Under **Permissions policies**, you should see:

```
iampolicy_siva
```

Status: Attached

<figure><img src=".gitbook/assets/image (41).png" alt=""><figcaption><p>IAM - Role Attached</p></figcaption></figure>

***

## Method 2: Attach Policy Using AWS CLI (Recommended for DevOps)

***

### How to configure AWS CLI?

On `aws-client` host, retrieve credentials:

```bash
showcreds
```

Then configure:

```bash
aws configure
```

Enter:

```bash
AWS Access Key ID:
AWS Secret Access Key:
Default region name: us-east-1
Default output format: json
```

***

### How to attach the policy via CLI?

Run:

```bash
aws iam attach-user-policy \
    --user-name iamuser_siva \
    --policy-arn arn:aws:iam::090742440071:policy/iampolicy_siva \
    --region us-east-1
```

***

### Expected Output

If successful, there will be **no output** returned.

***

### How to Verify via CLI?

```bash
aws iam list-attached-user-policies \
    --user-name iamuser_siva \
    --region us-east-1
```

#### Expected Output:

```json
{
  "AttachedPolicies": [
    {
      "PolicyName": "iampolicy_siva",
      "PolicyArn": "arn:aws:iam::090742440071:policy/iampolicy_siva"
    }
  ]
}
```

***

## Common Errors & Troubleshooting

***

### Error 1: NoSuchEntity

```bash
An error occurred (NoSuchEntity) when calling the AttachUserPolicy operation:
The user with name iamuser_siva cannot be found.
```

#### Cause:

* User does not exist
* Wrong region (IAM is global, but CLI region must still be set properly)

#### Fix:

Verify user exists:

```bash
aws iam get-user --user-name iamuser_siva
```

***

### Error 2: Policy Not Found

```bash
An error occurred (NoSuchEntity) when calling the AttachUserPolicy operation:
Policy arn not found
```

#### Cause:

Incorrect ARN or policy name.

#### Fix:

List policies:

```bash
aws iam list-policies --scope Local
```

***

### Error 3: AccessDenied

```bash
An error occurred (AccessDenied) when calling the AttachUserPolicy operation:
User is not authorized to perform: iam:AttachUserPolicy
```

#### Cause:

Logged-in IAM user lacks permission.

#### Fix:

Ensure your lab user has IAM admin privileges.

***

## Final Outcome

* IAM user `iamuser_siva` exists
* IAM policy `iampolicy_siva` exists
* Policy successfully attached
* Verified via console or CLI
* Resources used in `us-east-1`

***

## Final CLI Command Summary

```bash
aws configure
aws iam attach-user-policy --user-name iamuser_siva \
--policy-arn arn:aws:iam::090742440071:policy/iampolicy_siva \
--region us-east-1

aws iam list-attached-user-policies --user-name iamuser_siva
```

<figure><img src=".gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

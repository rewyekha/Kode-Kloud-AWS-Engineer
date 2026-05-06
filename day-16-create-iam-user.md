# Day 16: Create IAM User

Create an IAM user named `iamuser_anita`.

**Notes:**

* Create the resources only in `us-east-1` region.

---

## Solution

### Step 1: Configure AWS CLI

```bash
aws configure
```

Enter the credentials:

```
AWS Access Key ID: <your access key>
AWS Secret Access Key: <your secret key>
Default region name: us-east-1
Default output format: json
```

### Step 2: Create IAM User

```bash
aws iam create-user --user-name iamuser_anita
```

Example output:

```json
{
    "User": {
        "Path": "/",
        "UserName": "iamuser_anita",
        "UserId": "AIDA6EMGZHHVH3GTRPGFE",
        "Arn": "arn:aws:iam::971482151402:user/iamuser_anita",
        "CreateDate": "2026-02-27T04:40:40Z"
    }
}
```

### Step 3: Optional — Attach Managed Policy

```bash
aws iam attach-user-policy \
    --user-name iamuser_anita \
    --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

> **Note:** In this lab environment, permission may be **denied**. This is expected if your account lacks `iam:AttachUserPolicy`.

### Step 4: Verify IAM User

```bash
aws iam get-user --user-name iamuser_anita
```

Example output:

```json
{
    "User": {
        "Path": "/",
        "UserName": "iamuser_anita",
        "UserId": "AIDA6EMGZHHVH3GTRPGFE",
        "Arn": "arn:aws:iam::971482151402:user/iamuser_anita",
        "CreateDate": "2026-02-27T04:40:40Z"
    }
}
```

---

## Task Completion

* IAM user **`iamuser_anita`** created successfully.
* User verification completed via CLI.
* Policy attachment may be restricted due to lab permissions.

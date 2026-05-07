# Day 16: Create IAM User

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

When establishing infrastructure on the AWS cloud, Identity and Access Management (IAM) is among the first and most critical services to configure. IAM facilitates the creation and management of user accounts, groups, roles, policies, and other access controls. The Nautilus DevOps team is currently in the process of configuring these resources and has outlined the following requirements:

For this task, create an IAM user named `iamuser_anita`.\

Use below given AWS Credentials: (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

| Console URL | https://<aws-account>.signin.aws.amazon.com/console?region=us-east-1 |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Username    | <REDACTED_USERNAME> |
| Password    | <REDACTED_PASSWORD> |
| Start Time  | Fri Feb 27 04:39:39 UTC 2026                                                                                                               |
| End Time    | Fri Feb 27 05:39:39 UTC 2026                                                                                                               |

`Notes:`

* Create the resources only in `us-east-1` region.
* To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:\
  ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)

---

## Day 16: Create IAM User in AWS

### Objective

The Nautilus DevOps Team needed to create an **IAM user** in AWS to manage access for the DevOps team. IAM allows you to securely control access to AWS services and resources.

**Task Requirements:**

* Create an IAM user named `iamuser_anita`.
* Ensure the user can log in to the AWS Management Console.
* Use AWS region `us-east-1`.
* Optional: Attach administrative or required policies (if permissions allow).

---

### Environment & Credentials

* **AWS Console URL:** [https://<aws-account>.signin.aws.amazon.com/console?region=us-east-1](https://<aws-account>.signin.aws.amazon.com/console?region=us-east-1)
* **Username:** `<REDACTED_USERNAME>`
* **Password:** `<REDACTED_PASSWORD>`
* **Region:** us-east-1

> Note: The account used has limited permissions and may not allow attaching managed policies.

---

### Solution Steps

#### 1. Configure AWS CLI

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

---

#### 2. Create IAM User

```bash
aws iam create-user --user-name iamuser_anita
```

**Example Output:**

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

 Confirms the user was successfully created.

---

#### 3. Create Login Profile (Console Access)

```bash
aws iam create-login-profile \
    --user-name iamuser_anita \
    --password <REDACTED_PASSWORD> \
    --password-reset-required
```

* Forces user to **reset password at first login**.
* Allows access to AWS Management Console.

---

#### 4. Optional: Attach Managed Policy

```bash
aws iam attach-user-policy \
    --user-name iamuser_anita \
    --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

>  **Note:** In this lab environment, permission may be **denied**. This is normal and expected if your account lacks `iam:AttachUserPolicy`.

---

#### 5. Verify IAM User

```bash
aws iam get-user --user-name iamuser_anita
```

**Example Output:**

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

* Confirms the user exists in **us-east-1**.

---

###  Task Completion

* IAM User **`iamuser_anita`** created successfully.
* Login profile configured for AWS Console access.
* User verification completed via CLI.
* Policy attachment may be restricted due to lab permissions.

> The user can now be used for console login or programmatic access (API/CLI) with the credentials configured.

---

---

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

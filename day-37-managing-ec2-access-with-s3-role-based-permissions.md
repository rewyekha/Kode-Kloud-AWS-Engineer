# Day 37: Managing EC2 Access with S3 Role-based Permissions

## Nautilus DevOps — EC2 to S3 Integration Lab

**AWS Region:** us-east-1 **Account ID:** 254597876252 **Date:** April 4, 2026

***

### Table of Contents

1. [Lab Task Overview](https://claude.ai/chat/85f5b2d7-5dbe-46f3-abb7-eb5d74f7a104#lab-task-overview)
2. [Environment Details](https://claude.ai/chat/85f5b2d7-5dbe-46f3-abb7-eb5d74f7a104#environment-details)
3. [Step 1 — Verify the EC2 Instance](https://claude.ai/chat/85f5b2d7-5dbe-46f3-abb7-eb5d74f7a104#step-1--verify-the-ec2-instance)
4. [Step 2 — Create SSH Key Pair and Authorize on EC2](https://claude.ai/chat/85f5b2d7-5dbe-46f3-abb7-eb5d74f7a104#step-2--create-ssh-key-pair-and-authorize-on-ec2)
5. [Step 3 — Create a Private S3 Bucket](https://claude.ai/chat/85f5b2d7-5dbe-46f3-abb7-eb5d74f7a104#step-3--create-a-private-s3-bucket)
6. [Step 4 — Create IAM Policy, Role, and Attach to EC2](https://claude.ai/chat/85f5b2d7-5dbe-46f3-abb7-eb5d74f7a104#step-4--create-iam-policy-role-and-attach-to-ec2)
7. [Step 5 — Test S3 Access from the EC2 Instance](https://claude.ai/chat/85f5b2d7-5dbe-46f3-abb7-eb5d74f7a104#step-5--test-s3-access-from-the-ec2-instance)
8. [Lab Completion Summary](https://claude.ai/chat/85f5b2d7-5dbe-46f3-abb7-eb5d74f7a104#lab-completion-summary)

***

### Lab Task Overview

This lab covers the end-to-end setup of an EC2-to-S3 integration on AWS. The Nautilus DevOps team is required to configure an EC2 instance to securely interact with a private S3 bucket using IAM roles, without embedding any credentials on the instance.

#### Objectives

1. Identify the existing EC2 instance named `datacenter-ec2`.
2. Create a new SSH key pair on the `aws-client` host and add the public key to the `root` user on the EC2 instance.
3. Create a private S3 bucket named `datacenter-s3-254597876252`.
4. Create an IAM policy granting `s3:PutObject`, `s3:ListBucket`, and `s3:GetObject` on the bucket.
5. Create an IAM role named `datacenter-role`, attach the policy, and associate it with the EC2 instance.
6. SSH into the EC2 instance as `root` and validate S3 upload and list operations.

***

### Environment Details

| Parameter         | Value                                                               |
| ----------------- | ------------------------------------------------------------------- |
> **Note:** Run `showcreds` on the `aws-client` host to retrieve temporary AWS credentials, then configure the AWS CLI using `aws configure`.
| IAM Username      | kk\_labs\_user\_214087                                              |
| Region            | us-east-1                                                           |
| EC2 Instance Name | datacenter-ec2                                                      |
| Instance ID       | i-0d54d8ade5d07fd8b                                                 |
| Instance Type     | t2.micro                                                            |
| Public IP         | 18.212.149.103                                                      |
| Private IP        | 172.31.39.150                                                       |
| Availability Zone | us-east-1a                                                          |
| S3 Bucket         | datacenter-s3-254597876252                                          |
| IAM Role          | datacenter-role                                                     |
| IAM Policy        | datacenter-s3-policy                                                |

***

### Step 1 — Verify the EC2 Instance

Before proceeding, confirm that the `datacenter-ec2` instance exists and is in a running state. Run the following AWS CLI command on the `aws-client` host.

#### Command

```bash
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].[InstanceId,Tags[?Key==`Name`].Value | [0],State.Name,InstanceType,PublicIpAddress,PrivateIpAddress,Placement.AvailabilityZone]' \
  --output table
```

#### Output

```
-------------------------------------------------------------------------------------------------------------------
|                                                DescribeInstances                                                |
+---------------------+-----------------+----------+-----------+-----------------+-----------------+--------------+
|  i-0d54d8ade5d07fd8b|  datacenter-ec2 |  running |  t2.micro |  18.212.149.103 |  172.31.39.150  |  us-east-1a  |
+---------------------+-----------------+----------+-----------+-----------------+-----------------+--------------+
```

> The instance is confirmed running. Instance ID `i-0d54d8ade5d07fd8b` will be used in all subsequent steps.

***

### Step 2 — Create SSH Key Pair and Authorize on EC2

A new RSA key pair is generated on the `aws-client` host. The resulting public key is then injected into the `root` user's `authorized_keys` file on the EC2 instance to enable passwordless SSH access.

#### 2.1 — Generate the Key Pair on aws-client

```bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa -N ""
cat ~/.ssh/id_rsa.pub
```

#### Output

```bash
Generating public/private rsa key pair.
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:GlFrqJ4BqmMCUAWQYxwl4AyWlNhO9lWdUsuAJpRHJY4 root@aws-client
The key's randomart image is:
+---[RSA 2048]----+
|OOB=ooo+=o..     |
|O*= oo=+.+o.     |
|oB..E=+ o.o      |
|...... o         |
|o   o . S        |
|o  . o o         |
|+.  o .          |
|o.               |
|                 |
+----[SHA256]-----+

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDg9jZC65eWSqRHDM47RJFPKPp/fg12uxNtYDz/GWuJ6498i2m69M7lT
CwncMAjmFq64GXKjGvt2wrjfv0m4v4ncJsVdXe6KmEYY+e2yGAH7H+QXswhIgs2QFMdDH7nRPW/J0WyjqXMOwNe4gv1y
d22F+7FHJAAc1MQHJhA9344s85po0n/RxIdjx5a1UP7TAj7ICNPDZUI8HvXDu8HTSa7RrDgPIY6ywSpMEi5BpiNDFuyU
CT5Z6WsCOedb+fXKVPfTP1Pkih9soHlebxYPUryqTAPExx9ZoAalxHlYgmQ+jyDXOVz8Ca/8vtZr8295dVuMYw76Oimi
OkSNqzvagyF root@aws-client
```

#### 2.2 — Store the Public Key in a Variable

```bash
PUB_KEY=$(cat ~/.ssh/id_rsa.pub)
```

#### 2.3 — Connect to EC2 via AWS Session Manager and Inject the Public Key

Connect to the EC2 instance using the AWS Management Console Session Manager (EC2 > Connect > Session Manager). Once connected as `ubuntu`, run the following commands to authorize the public key for the `root` user.

```bash
# Connected to EC2 instance as ubuntu via Session Manager
sudo mkdir -p /root/.ssh

echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDg9jZC65eWSqRHDM47RJFPKPp/fg12uxNtYDz/GWuJ6498i2m69M7lTCwncMAjmFq64GXKjGvt2wrjfv0m4v4ncJsVdXe6KmEYY+e2yGAH7H+QXswhIgs2QFMdDH7nRPW/J0WyjqXMOwNe4gv1yd22F+7FHJAAc1MQHJhA9344s85po0n/RxIdjx5a1UP7TAj7ICNPDZUI8HvXDu8HTSa7RrDgPIY6ywSpMEi5BpiNDFuyUCT5Z6WsCOedb+fXKVPfTP1Pkih9soHlebxYPUryqTAPExx9ZoAalxHlYgmQ+jyDXOVz8Ca/8vtZr8295dVuMYw76OimiOkSNqzvagyF root@aws-client" \
  | sudo tee /root/.ssh/authorized_keys

sudo chmod 700 /root/.ssh
sudo chmod 600 /root/.ssh/authorized_keys
```

#### Session Manager — EC2 Connection Banner

```
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.5.0-1022-aws x86_64)

  System load:  0.01              Processes:             99
  Usage of /:   29.0% of 7.57GB   Users logged in:       0
  Memory usage: 24%               IPv4 address for eth0: 172.31.39.150
  Swap usage:   0%

ubuntu@ip-172-31-39-150:~$
```

> The public key is now present in `/root/.ssh/authorized_keys` on the EC2 instance, enabling direct SSH access as `root` from the `aws-client` host.

***

### Step 3 — Create a Private S3 Bucket

Create the S3 bucket in `us-east-1` and immediately apply a public access block to ensure it remains fully private. No objects in this bucket will ever be publicly accessible.

#### Command

```bash
# Create the bucket
aws s3api create-bucket \
  --bucket datacenter-s3-254597876252 \
  --region us-east-1

# Block all public access
aws s3api put-public-access-block \
  --bucket datacenter-s3-254597876252 \
  --public-access-block-configuration \
  "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

#### Output

```json
{
    "Location": "/datacenter-s3-254597876252"
}
```

> The bucket was created successfully. The public access block prevents any accidental public exposure through ACLs or bucket policies.

***

### Step 4 — Create IAM Policy, Role, and Attach to EC2

This step involves four sub-tasks: creating the IAM policy document, creating the IAM role with an EC2 trust relationship, attaching the policy to the role, and associating the instance profile with the EC2 instance.

#### 4.1 — Create the IAM Policy

```bash
cat > /tmp/s3-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject","s3:GetObject","s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::datacenter-s3-254597876252",
        "arn:aws:s3:::datacenter-s3-254597876252/*"
      ]
    }
  ]
}
EOF

aws iam create-policy \
  --policy-name datacenter-s3-policy \
  --policy-document file:///tmp/s3-policy.json
```

#### Output

```json
{
    "Policy": {
        "PolicyName": "datacenter-s3-policy",
        "PolicyId": "ANPATWRZXJIOHMIJOQ34Y",
        "Arn": "arn:aws:iam::254597876252:policy/datacenter-s3-policy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2026-04-04T04:01:10Z",
        "UpdateDate": "2026-04-04T04:01:10Z"
    }
}
```

#### 4.2 — Create the IAM Role with EC2 Trust Policy

```bash
cat > /tmp/trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

aws iam create-role \
  --role-name datacenter-role \
  --assume-role-policy-document file:///tmp/trust-policy.json
```

#### Output

```json
{
    "Role": {
        "Path": "/",
        "RoleName": "datacenter-role",
        "RoleId": "AROATWRZXJIOBSVYH55MB",
        "Arn": "arn:aws:iam::254597876252:role/datacenter-role",
        "CreateDate": "2026-04-04T04:01:11Z",
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

#### 4.3 — Attach the Policy to the Role

```bash
aws iam attach-role-policy \
  --role-name datacenter-role \
  --policy-arn arn:aws:iam::254597876252:policy/datacenter-s3-policy
```

No output is returned on success.

#### 4.4 — Create Instance Profile and Associate with EC2

```bash
# Create instance profile
aws iam create-instance-profile \
  --instance-profile-name datacenter-role

# Add role to instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name datacenter-role \
  --role-name datacenter-role

# Attach instance profile to the EC2 instance
aws ec2 associate-iam-instance-profile \
  --instance-id i-0d54d8ade5d07fd8b \
  --iam-instance-profile Name=datacenter-role
```

#### Output

```json
{
    "InstanceProfile": {
        "Path": "/",
        "InstanceProfileName": "datacenter-role",
        "InstanceProfileId": "AIPATWRZXJIOML2BAO52L",
        "Arn": "arn:aws:iam::254597876252:instance-profile/datacenter-role",
        "CreateDate": "2026-04-04T04:01:13Z",
        "Roles": []
    }
}

{
    "IamInstanceProfileAssociation": {
        "AssociationId": "iip-assoc-00d2131555526902d",
        "InstanceId": "i-0d54d8ade5d07fd8b",
        "IamInstanceProfile": {
            "Arn": "arn:aws:iam::254597876252:instance-profile/datacenter-role",
            "Id": "AIPATWRZXJIOML2BAO52L"
        },
        "State": "associating"
    }
}
```

#### 4.5 — Verify the Association

```bash
aws ec2 describe-iam-instance-profile-associations \
  --query 'IamInstanceProfileAssociations[*].[InstanceId,IamInstanceProfile.Arn,State]' \
  --output table
```

#### Output

```
-----------------------------------------------------------------------------------------------------
|                              DescribeIamInstanceProfileAssociations                               |
+---------------------+--------------------------------------------------------------+--------------+
|  i-0d54d8ade5d07fd8b|  arn:aws:iam::254597876252:instance-profile/datacenter-role  |  associated  |
+---------------------+--------------------------------------------------------------+--------------+
```

> The IAM role `datacenter-role` is now in `associated` state on the EC2 instance. The instance will use this role's credentials automatically via the instance metadata service (IMDS). No static credentials are required.

***

### Step 5 — Test S3 Access from the EC2 Instance

With the IAM role attached, SSH into the EC2 instance as `root` using the key pair generated in Step 2. Upload a test file and list the bucket contents to confirm that the permissions are working correctly.

#### 5.1 — SSH into the EC2 Instance as root

```bash
ssh -i ~/.ssh/id_rsa -o StrictHostKeyChecking=no root@18.212.149.103
```

#### Output

```
Warning: Permanently added '18.212.149.103' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.5.0-1022-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System load:  0.0               Processes:             108
  Usage of /:   29.0% of 7.57GB   Users logged in:       1
  Memory usage: 25%               IPv4 address for eth0: 172.31.39.150
  Swap usage:   0%

root@ip-172-31-39-150:~#
```

#### 5.2 — Upload a File to the S3 Bucket

```bash
echo "test data from datacenter-ec2" > /tmp/testfile.txt

aws s3 cp /tmp/testfile.txt s3://datacenter-s3-254597876252/
```

#### Output

```
upload: ../tmp/testfile.txt to s3://datacenter-s3-254597876252/testfile.txt
```

#### 5.3 — List the Bucket Contents

```bash
aws s3 ls s3://datacenter-s3-254597876252/
```

#### Output

```
2026-04-04 04:02:45         30 testfile.txt
```

> The file `testfile.txt` was successfully uploaded and listed. The EC2 instance is accessing S3 exclusively through the attached IAM role, with no hardcoded credentials on the instance.

***

### Lab Completion Summary

| Step | Task                                                       | Status   | Details                                       |
| ---- | ---------------------------------------------------------- | -------- | --------------------------------------------- |
| 1    | Verified EC2 instance `datacenter-ec2` is running          | Complete | Instance ID confirmed                         |
| 2    | Generated SSH key pair and added to root `authorized_keys` | Complete | `id_rsa` / `id_rsa.pub` created on aws-client |
| 3    | Created private S3 bucket `datacenter-s3-254597876252`     | Complete | Public access fully blocked                   |
| 4    | Created IAM policy `datacenter-s3-policy`                  | Complete | `PutObject`, `GetObject`, `ListBucket`        |
| 5    | Created IAM role `datacenter-role` with EC2 trust          | Complete | `ec2.amazonaws.com` principal                 |
| 6    | Attached policy to role and associated role to EC2         | Complete | State: `associated`                           |
| 7    | SSH'd into EC2 as `root` and uploaded file to S3           | Complete | `testfile.txt` uploaded                       |
| 8    | Listed S3 bucket contents from EC2                         | Complete | `testfile.txt` confirmed present              |

***

All lab objectives have been met. The EC2 instance `datacenter-ec2` is successfully integrated with the private S3 bucket `datacenter-s3-254597876252` via IAM role-based access. No static credentials were used or stored on the instance.

<figure><img src=".gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

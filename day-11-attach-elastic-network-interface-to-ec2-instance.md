# Day 11: Attach Elastic Network Interface to EC2 Instance

The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.

An instance named `datacenter-ec2` and an elastic network interface named `datacenter-eni` already exists in `us-east-1` region.

* Attach the `datacenter-eni` network interface to the `datacenter-ec2` instance.
* Make sure status is `attached` before submitting the task.

Please make sure instance initialisation has been completed before submitting this task.

Use below given AWS Credentials. (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

> **Note:** Run `showcreds` on the `aws-client` host to retrieve temporary AWS credentials, then configure the AWS CLI using `aws configure`.

**Notes:**

* Create the resources only in `us-east-1` region.

<figure><img src=".gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

## Attaching an Existing ENI to an EC2 Instance (GUI & CLI)

The Nautilus DevOps team is migrating services to AWS. This guide explains how to **attach an existing Elastic Network Interface (ENI) `datacenter-eni` to an EC2 instance `datacenter-ec2`** in the **us-east-1** region using **both GUI and CLI**.

> **Important Notes:**
>
> * Only use **us-east-1** region.
> * Ensure the EC2 instance is **fully initialized** before attaching the ENI.
> * Wait for the ENI status to show **in-use** before submitting the task.

***

### AWS Credentials

| Field       | Value                                                                              |
| ----------- | ---------------------------------------------------------------------------------- |
> **Note:** Run `showcreds` on the `aws-client` host to retrieve temporary AWS credentials, then configure the AWS CLI using `aws configure`.

> You can also retrieve credentials via `showcreds` command on the `aws-client` host if using CLI.

***

### Method 1: Using AWS Console (GUI)

#### Step 1: Log in and Verify Region

1. Open [AWS Console](the AWS Management Console URL).
2. Enter **Username** and **Password**.
3. Ensure the **region is set to `us-east-1`** (top-right corner).

***

#### Step 2: Verify EC2 Instance Initialization

1. Go to **Services → EC2 → Instances**.
2. Find `datacenter-ec2`.
3. Check **Instance State** → should be **running**.
4. Confirm **Status Checks** show **2/2 checks passed**.

***

#### Step 3: Attach ENI

1. Navigate to **Services → EC2 → Network Interfaces**.
2. Find `datacenter-eni`.
3. Select the ENI → **Actions → Attach**.
4. Choose **Instance** → `datacenter-ec2`.
5. Click **Attach**.

***

#### Step 4: Confirm Attachment

1. Go to **EC2 → Instances → datacenter-ec2 → Networking tab**.
2. Ensure `datacenter-eni` is listed.
3. Verify **Status** shows **in-use**.

***

### Method 2: Using AWS CLI

> **Prerequisite:** AWS CLI installed and configured with credentials for `us-east-1`.

#### Step 1: Verify Instance Initialization

```bash
aws ec2 describe-instances \
    --instance-ids datacenter-ec2 \
    --region us-east-1 \
    --query 'Reservations[*].Instances[*].[InstanceId,State.Name,StateReason]'
```

* Ensure **State.Name** is `running`.

***

#### Step 2: Attach ENI to EC2 Instance

```bash
aws ec2 attach-network-interface \
    --network-interface-id datacenter-eni \
    --instance-id datacenter-ec2 \
    --device-index 1 \
    --region us-east-1
```

* **`device-index`**: Choose 1 if the instance has its primary ENI already attached.

***

#### Step 3: Verify ENI Attachment

```bash
aws ec2 describe-network-interfaces \
    --network-interface-ids datacenter-eni \
    --region us-east-1 \
    --query 'NetworkInterfaces[*].[Status,Attachment.InstanceId]'
```

* Confirm **Status** is `in-use` and **Attachment.InstanceId** is `datacenter-ec2`.

***

#### Step 4: Confirm Network Interface on Instance

```bash
aws ec2 describe-instances \
    --instance-ids datacenter-ec2 \
    --region us-east-1 \
    --query 'Reservations[*].Instances[*].NetworkInterfaces[*].NetworkInterfaceId'
```

* `datacenter-eni` should appear in the list.

***

### Notes

* Only attach resources in **us-east-1**.
* Do not detach other ENIs unless required.
* Ensure EC2 instance initialization is complete before submitting the task.

# Day 2: Security Group

The Nautilus DevOps team is migrating infrastructure to AWS. For this task, create a security group under the default VPC with the following requirements:

* Name of the security group is `xfusion-sg`.
* The description must be `Security group for Nautilus App Servers`
* Add an inbound rule of type `HTTP`, with port range of `80` and source CIDR `0.0.0.0/0`.
* Add an inbound rule of type `SSH`, with port range of `22` and source CIDR `0.0.0.0/0`.

<figure><img src=".gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

**Notes:**

* You are logged into the **aws-client** host.
* AWS credentials are already configured using `showcreds`.
* Region is already set to `us-east-1`.

---

## AWS CLI Steps

### Step 1: Verify AWS CLI Configuration

```bash
aws sts get-caller-identity
```

### Step 2: Get the Default VPC ID

You must create the security group inside the **default VPC**.

```bash
VPC_ID=$(aws ec2 describe-vpcs \
  --filters Name=isDefault,Values=true \
  --query "Vpcs[0].VpcId" \
  --output text)

echo $VPC_ID
```

### Step 3: Create the Security Group

Create a security group named **xfusion-sg** with the given description.

```bash
SG_ID=$(aws ec2 create-security-group \
  --group-name xfusion-sg \
  --description "Security group for Nautilus App Servers" \
  --vpc-id $VPC_ID \
  --query "GroupId" \
  --output text)

echo $SG_ID
```

### Step 4: Add Inbound Rule for HTTP (Port 80)

Allow HTTP access from anywhere.

```bash
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

### Step 5: Add Inbound Rule for SSH (Port 22)

Allow SSH access from anywhere.

```bash
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

### Step 6: Verify the Security Group

Confirm that the rules were added correctly.

```bash
aws ec2 describe-security-groups \
  --group-ids $SG_ID
```

---

## Result

* Security Group Name: `xfusion-sg`
* VPC: Default VPC
* Inbound Rules:
  * HTTP (80) -> `0.0.0.0/0`
  * SSH (22) -> `0.0.0.0/0`

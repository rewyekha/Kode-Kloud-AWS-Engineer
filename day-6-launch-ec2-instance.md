# Day 6: Launch EC2 Instance

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units.

For this task, create an EC2 instance with following requirements:

1\) The name of the instance must be `xfusion-ec2`.

2\) You can use the `Amazon Linux` AMI to launch this instance.

3\) The Instance type must be `t2.micro`.

4\) Create a new RSA key pair named `xfusion-kp`.

5\) Attach the default (available by default) security group.

> **Note:** Run `showcreds` on the `aws-client` host to retrieve temporary AWS credentials, then configure the AWS CLI using `aws configure`.

**Notes:**

* Create the instance in `us-east-1` region.
Below is a **clear, step-by-step AWS CLI solution** to create the required **EC2 instance** from the **aws-client host**, exactly matching the task constraints.

***

### Task Requirements Recap

| Requirement    | Value                           |
| -------------- | ------------------------------- |
| Instance name  | `xfusion-ec2`                   |
| AMI            | Amazon Linux                    |
| Instance type  | `t2.micro`                      |
| Key pair       | New RSA key pair → `xfusion-kp` |
| Security group | Default (pre-existing)          |
| Region         | `us-east-1`                     |
| Credentials    | Via `showcreds`                 |

***

### Step 1: Load AWS Credentials

On the **aws-client** host:

```bash
showcreds
```

This exports the temporary AWS credentials into your shell environment.

* Do **not** manually configure keys.

***

### Step 2: Set AWS Region

```bash
aws configure set region us-east-1
```

Verify:

```bash
aws configure get region
```

Expected:

```
us-east-1
```

***

### Step 3: Create RSA Key Pair

```bash
aws ec2 create-key-pair \
  --key-name xfusion-kp \
  --key-type rsa \
  --query 'KeyMaterial' \
  --output text > xfusion-kp.pem
```

Set correct permissions:

```bash
chmod 400 xfusion-kp.pem
```

* RSA key pair `xfusion-kp` created
* Private key saved locally

***

### Step 4: Get Latest Amazon Linux AMI (Recommended Way)

Fetch the latest Amazon Linux 2 AMI using AWS SSM:

```bash
AMI_ID=$(aws ssm get-parameter \
  --name /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 \
  --query "Parameter.Value" \
  --output text)
```

Verify:

```bash
echo $AMI_ID
```

***

### Step 5: Get Default Security Group ID

```bash
SG_ID=$(aws ec2 describe-security-groups \
  --filters Name=group-name,Values=default \
  --query "SecurityGroups[0].GroupId" \
  --output text)
```

***

### Step 6: Launch the EC2 Instance

```bash
aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t2.micro \
  --key-name xfusion-kp \
  --security-group-ids $SG_ID \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-ec2}]' \
  --count 1
```

Expected result:

* Instance launches in **running** or **pending** state
* Tagged as `xfusion-ec2`

***

### Step 7: Verify the Instance

```bash
aws ec2 describe-instances \
  --filters Name=tag:Name,Values=xfusion-ec2 \
  --query "Reservations[*].Instances[*].[InstanceId,InstanceType,State.Name]" \
  --output table
```

Expected output:

```
-----------------------------------------
|      DescribeInstances                 |
+-------------+------------+-------------+
| i-0abcd123  | t2.micro  | running     |
+-------------+------------+-------------+
```

***

### Final Checklist

* Instance name: **xfusion-ec2**
* AMI: **Amazon Linux**
* Instance type: **t2.micro**
* Key pair: **xfusion-kp (RSA)**
* Security group: **default**
* Region: **us-east-1**

***

### Conclusion

The EC2 instance has been **successfully created** following AWS best practices and the Nautilus DevOps team’s **incremental migration strategy**.


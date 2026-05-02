# Day 21: Setting Up an EC2 Instance with an Elastic IP for Application Hosting

The Nautilus DevOps Team has received a new request from the Development Team to set up a new EC2 instance. This instance will be used to host a new application that requires a stable IP address. To ensure that the instance has a consistent public IP, an Elastic IP address needs to be associated with it. The instance will be named `xfusion-ec2`, and the Elastic IP will be named `xfusion-eip`. This setup will help the Development Team to have a reliable and consistent access point for their application.

Create an EC2 instance named `xfusion-ec2` using any linux AMI like ubuntu, the Instance type must be `t2.micro` and associate an `Elastic IP` address with this instance, name it as `xfusion-eip`.

> **Note:** Run `showcreds` on the `aws-client` host to retrieve temporary AWS credentials, then configure the AWS CLI using `aws configure`.

**Notes:**

* Create the resources only in `us-east-1` region.
 ***

## Create EC2 Instance and Associate Elastic IP

***

## Question

The Nautilus DevOps Team received a request from the Development Team to:

* Create an EC2 instance named **`xfusion-ec2`**
* Use a Linux AMI (Ubuntu preferred)
* Instance type must be **t2.micro**
* Region must be **us-east-1**
* Allocate and associate an Elastic IP named **`xfusion-eip`**

This ensures the instance has a stable public IP for application access.

***

## Solution Using AWS CLI

> Region: `us-east-1`

***

### 1. Get Latest Ubuntu AMI

```bash
aws ec2 describe-images \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-*-amd64-server-*" \
  --query 'Images[*].[ImageId,Name]' \
  --region us-east-1
```

#### Output

```json
[
    [
        "ami-04680790a315cd58d",
        "ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-20260218"
    ]
]
```

Selected AMI:

```
ami-04680790a315cd58d
```

***

### 2. Launch EC2 Instance

```bash
aws ec2 run-instances \
  --image-id ami-04680790a315cd58d \
  --count 1 \
  --instance-type t2.micro \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-ec2}]' \
  --region us-east-1
```

#### Output (Important Details)

```json
{
    "Instances": [
        {
            "InstanceId": "i-008cf408fd36ccb3b",
            "InstanceType": "t2.micro",
            "State": {
                "Name": "pending"
            }
        }
    ]
}
```

***

### 3. Wait Until Instance is Running

```bash
aws ec2 describe-instances \
  --instance-ids i-008cf408fd36ccb3b \
  --query "Reservations[*].Instances[*].State.Name" \
  --region us-east-1
```

#### Output

```json
[
    [
        "running"
    ]
]
```

***

### 4. Allocate Elastic IP

```bash
aws ec2 allocate-address \
  --domain vpc \
  --region us-east-1
```

#### Output

```json
{
    "AllocationId": "eipalloc-0b9e1b12ae3962144",
    "PublicIp": "54.173.34.178"
}
```

***

### 5. Tag Elastic IP as xfusion-eip

```bash
aws ec2 create-tags \
  --resources eipalloc-0b9e1b12ae3962144 \
  --tags Key=Name,Value=xfusion-eip \
  --region us-east-1
```

(No output = success)

***

### 6. Associate Elastic IP to Instance

```bash
aws ec2 associate-address \
  --instance-id i-008cf408fd36ccb3b \
  --allocation-id eipalloc-0b9e1b12ae3962144 \
  --region us-east-1
```

#### Output

```json
{
    "AssociationId": "eipassoc-03b6e9e1a03ea2992"
}
```

***

### 7. Final Verification

#### Check Instance Public IP

```bash
aws ec2 describe-instances \
  --instance-ids i-008cf408fd36ccb3b \
  --query "Reservations[*].Instances[*].[InstanceId,PublicIpAddress]" \
  --region us-east-1
```

#### Output

```json
[
    [
        [
            "i-008cf408fd36ccb3b",
            "54.173.34.178"
        ]
    ]
]
```

***

#### Check Elastic IP Details

```bash
aws ec2 describe-addresses --region us-east-1
```

#### Output

```json
{
    "Addresses": [
        {
            "AllocationId": "eipalloc-0b9e1b12ae3962144",
            "InstanceId": "i-008cf408fd36ccb3b",
            "PublicIp": "54.173.34.178",
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "xfusion-eip"
                }
            ]
        }
    ]
}
```

***

## Solution Using AWS Console (GUI)

***

### Step 1: Login to AWS Console

* Open AWS Console
* Select region: **US East (N. Virginia) – us-east-1**

***

### Step 2: Launch EC2 Instance

1. Go to **EC2 → Instances**
2. Click **Launch Instance**

#### Configure:

* **Name:** `xfusion-ec2`
* **AMI:** Ubuntu Server 22.04
* **Instance Type:** `t2.micro`
* **Key Pair:** Select or create
* **Network:** Default VPC
* **Auto-assign Public IP:** Enabled
* **Security Group:** Allow SSH (Port 22)

Click **Launch Instance**

***

### Step 3: Allocate Elastic IP

1. Go to **EC2 → Elastic IPs**
2. Click **Allocate Elastic IP**
3. Keep default settings
4. Click **Allocate**

***

### Step 4: Tag Elastic IP

1. Select the allocated Elastic IP
2. Click **Actions → Add/Edit Tags**
3. Add:

| Key  | Value       |
| ---- | ----------- |
| Name | xfusion-eip |

Save changes.

***

### Step 5: Associate Elastic IP

1. Select Elastic IP
2. Click **Actions → Associate Elastic IP**
3. Choose:
   * Resource Type: Instance
   * Instance: `xfusion-ec2`
4. Click **Associate**

***

## Final Validation Checklist

| Requirement                  | Status |
| ---------------------------- | ------ |
| Region us-east-1             |       |
| Ubuntu Linux AMI             |       |
| Instance Name xfusion-ec2    |       |
| Instance Type t2.micro       |       |
| Elastic IP allocated         |       |
| Elastic IP named xfusion-eip |       |
| Elastic IP associated        |       |
| Stable Public IP assigned    |       |

***

## Final Result

```
Instance Name: xfusion-ec2
Instance ID: i-008cf408fd36ccb3b
Instance Type: t2.micro
Elastic IP: 54.173.34.178
Elastic IP Name: xfusion-eip
Region: us-east-1
```

***

* Task Completed Successfully.

<figure><img src=".gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>


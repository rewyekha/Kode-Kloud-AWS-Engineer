# Day 10: Attach Elastic IP to EC2 Instance

The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.

There is an instance named `nautilus-ec2` and an elastic-ip named `nautilus-ec2-eip` in `us-east-1` region. Attach the `nautilus-ec2-eip` elastic-ip to the `nautilus-ec2` instance.

> **Note:** Run `showcreds` on the `aws-client` host to retrieve temporary AWS credentials, then configure the AWS CLI using `aws configure`.

**Notes:**

* Create the resources only in `us-east-1` region.

### Objective

Attach an existing **Elastic IP** named **`nautilus-ec2-eip`** to an existing **EC2 instance** named **`nautilus-ec2`** in the **`us-east-1`** region.

***

### Given Constraints (Important)

* **Region:** `us-east-1` only
* **EC2 Instance:** `nautilus-ec2` (already exists)
* **Elastic IP:** `nautilus-ec2-eip` (already allocated)
*  Do **NOT** create new resources

***

## Method 1: AWS CLI

#### Step 1: Login to aws-client host

```bash
ssh aws-client
```

***

#### Step 2: Load AWS credentials

```bash
showcreds
```

Copy:

* AWS\_ACCESS\_KEY\_ID
* AWS\_SECRET\_ACCESS\_KEY
* AWS\_SESSION\_TOKEN

***

#### Step 3: Configure AWS CLI

```bash
aws configure
```

Enter:

* **Access Key** → from `showcreds`
* **Secret Key** → from `showcreds`
* **Region** → `us-east-1`
* **Output format** → `json`

***

#### Step 4: Get EC2 Instance ID

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=nautilus-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text
```

* Example output:

```
i-0abc1234def567890
```

***

#### Step 5: Get Elastic IP Allocation ID

```bash
aws ec2 describe-addresses \
  --filters "Name=tag:Name,Values=nautilus-ec2-eip" \
  --query "Addresses[].AllocationId" \
  --output text
```

* Example output:

```
eipalloc-0123456789abcdef0
```

***

#### Step 6: Attach Elastic IP to EC2

```bash
aws ec2 associate-address \
  --instance-id i-0abc1234def567890 \
  --allocation-id eipalloc-0123456789abcdef0
```

* Successful execution returns an **AssociationId**.

***

#### (Optional) Verify Attachment

```bash
aws ec2 describe-addresses \
  --allocation-ids eipalloc-0123456789abcdef0
```

Expected:

* `InstanceId` → `nautilus-ec2`

***

## Method 2: AWS Console (GUI)

#### Step 1: Login to AWS Console

Open **AWS Management Console**

* **Region:** `us-east-1`
* Use provided **Username & Password**

***

#### Step 2: Navigate to Elastic IPs

```
Services → EC2 → Network & Security → Elastic IPs
```

***

#### Step 3: Select the Elastic IP

* Choose **`nautilus-ec2-eip`**
* Click **Actions → Associate Elastic IP address**

***

#### Step 4: Associate with EC2 Instance

* **Resource type:** Instance
* **Instance:** `nautilus-ec2`
* **Private IP:** (leave default)

Click **Associate**

![Image](https://www.turnkeylinux.org/files/images/01_Elastic_IP_addresses_EC2_Console.png)

![Image](https://media.tutorialsdojo.com/public/td-pc-lab-elastic-ip-steps-15-Aug-2024-image-12.png)

![Image](https://i0.wp.com/economizecloud.wpengine.com/wp-content/uploads/2023/11/ElasticIP.png?resize=721%2C561\&ssl=1)

***

#### Step 5: Verify

* Elastic IP status → **In use**
* Associated instance → **nautilus-ec2**

***

## Final Validation Checklist

* Elastic IP: `nautilus-ec2-eip`
* Instance: `nautilus-ec2`
* Region: `us-east-1`
* EIP status: **Associated / In use**

***

### What went wrong (very common mistake)

You ran this command:

```bash
aws ec2 describe-addresses \
  --allocation-ids eipassoc-0045976d3b4818dfa
```

But:

* `eipassoc-0045976d3b4818dfa` → **Association ID**
* `describe-addresses` expects → **Allocation ID** (`eipalloc-xxxx`)

That’s why AWS says:

```
InvalidAllocationID.NotFound
```

***

### Your attachment WAS successful

This output confirms success:

```json
{
  "AssociationId": "eipassoc-0045976d3b4818dfa"
}
```

If association failed, AWS would have thrown an error immediately.

***

### Correct way to verify (2 options)

#### Option 1: Verify using Allocation ID (recommended)

```bash
aws ec2 describe-addresses \
  --allocation-ids eipalloc-0ac22d845d7da377b
```

You should see:

* `"InstanceId": "i-066be603cea44b979"`
* `"AssociationId": "eipassoc-0045976d3b4818dfa"`

***

#### Option 2: Verify via EC2 instance

```bash
aws ec2 describe-instances \
  --instance-ids i-066be603cea44b979 \
  --query "Reservations[].Instances[].PublicIpAddress" \
  --output text
```

This should return the **Elastic IP address**, not a random public IP.

***

### Final Status

* Elastic IP attached successfully
* Instance: `nautilus-ec2`
* Region: `us-east-1`
* Task completed correctly


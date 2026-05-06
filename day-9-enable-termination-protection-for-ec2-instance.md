# Day 9: Enable Termination Protection for EC2 Instance

As part of the migration, the Nautilus DevOps team created an EC2 instance but forgot to enable termination protection. Termination protection prevents accidental deletion of critical EC2 instances.

An instance named `nautilus-ec2` already exists in `us-east-1` region. Enable `termination protection` for the same.

**Notes:**

* Create the resources only in `us-east-1` region.

---

## Method 1: AWS CLI

### Step 1: Log in to aws-client host

```bash
ssh aws-client
```

### Step 2: Load AWS credentials

```bash
showcreds
```

Copy the **AWS_ACCESS_KEY_ID**, **AWS_SECRET_ACCESS_KEY**, and **AWS_SESSION_TOKEN**.

### Step 3: Configure AWS CLI

```bash
aws configure
```

Enter:

* **AWS Access Key ID** -> from `showcreds`
* **AWS Secret Access Key** -> from `showcreds`
* **Default region name** -> `us-east-1`
* **Default output format** -> `json`

### Step 4: Get Instance ID of `nautilus-ec2`

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=nautilus-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text
```

Example output:

```
i-0abc12345def6789
```

### Step 5: Enable Termination Protection

```bash
aws ec2 modify-instance-attribute \
  --instance-id i-0abc12345def6789 \
  --disable-api-termination
```

If no output appears, the command was successful.

### Step 6: Verify Protection Status

```bash
aws ec2 describe-instance-attribute \
  --instance-id i-0abc12345def6789 \
  --attribute disableApiTermination
```

Expected output:

```json
{
  "DisableApiTermination": {
    "Value": true
  }
}
```

---

## Method 2: AWS Console (GUI)

### Step 1: Log in to the AWS Management Console

* **Region:** `us-east-1`

### Step 2: Navigate to EC2

```
Services -> EC2 -> Instances
```

### Step 3: Select the instance

* Find instance named **`nautilus-ec2`**
* Select the checkbox

### Step 4: Enable Termination Protection

```
Actions -> Instance settings -> Change termination protection
```

* Select **Enable**
* Click **Save**

<figure><img src=".gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

### Step 5: Verify

* Select the instance
* Go to **Details tab**
* Confirm:

```
Termination protection: Enabled
```

---

## Result

* Instance name: `nautilus-ec2`
* Region: `us-east-1`
* Termination protection: **Enabled**
* No new resources created

---

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=nautilus-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text
i-0854cd505f18ee159

aws ec2 modify-instance-attribute \
  --instance-id i-0854cd505f18ee159 \
  --disable-api-termination

aws ec2 describe-instance-attribute \
  --instance-id i-0854cd505f18ee159 \
  --attribute disableApiTermination
{
    "DisableApiTermination": {
        "Value": true
    },
    "InstanceId": "i-0854cd505f18ee159"
}
```

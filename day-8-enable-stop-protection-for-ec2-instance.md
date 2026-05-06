# Day 8: Enable Stop Protection for EC2 Instance

As part of the migration, the team created an EC2 instance where they need to enable stop protection.

There is an EC2 instance named `datacenter-ec2` under `us-east-1` region. Enable the `stop` protection for this instance.

**Notes:**

* Create the resources only in `us-east-1` region.

> Stop protection prevents the instance from being stopped via API, CLI, or Console.

---

## AWS CLI Steps

### Step 1: Configure AWS CLI

Run `showcreds` on the **aws-client** host and then configure AWS CLI:

```bash
aws configure
```

Provide:

* **AWS Access Key ID**: (from `showcreds`)
* **AWS Secret Access Key**: (from `showcreds`)
* **Default region name**: us-east-1
* **Default output format**: json

Verify:

```bash
aws sts get-caller-identity
```

### Step 2: Find the Instance ID for `datacenter-ec2`

```bash
aws ec2 describe-instances \
  --region us-east-1 \
  --filters "Name=tag:Name,Values=datacenter-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text
```

Example output:

```
i-0abc1234def567890
```

### Step 3: Enable Stop Protection on the Instance

Replace `<INSTANCE_ID>` with the value from the previous step.

```bash
aws ec2 modify-instance-attribute \
  --region us-east-1 \
  --instance-id <INSTANCE_ID> \
  --disable-api-stop
```

### Step 4: Verify Stop Protection Is Enabled

```bash
aws ec2 describe-instance-attribute \
  --region us-east-1 \
  --instance-id <INSTANCE_ID> \
  --attribute disableApiStop
```

Expected output:

```json
{
  "DisableApiStop": {
    "Value": true
  }
}
```

---

## Result

Stop protection is now enabled for **datacenter-ec2** in **us-east-1**.

---

## Console (GUI) Steps

1. **Log in to the AWS Management Console.**
2. **Go to EC2**
   * From the AWS Console home page, select **Services**
   * Click **EC2**
3. **Make sure you are in the correct region**
   * Top-right corner -> select **US East (N. Virginia)**
4. **Open Instances**
   * In the left navigation pane, click **Instances**
5. **Select the instance**
   * Locate the instance named **`datacenter-ec2`**
   * Select the checkbox next to the instance
6. **Enable Stop Protection**
   * With the instance selected, click **Actions**
   * Choose **Instance settings**
   * Click **Change stop protection**
   * Check **Enable stop protection**
   * Click **Save**
7. **Confirmation**
   * You should see a confirmation message that stop protection has been enabled.

The EC2 instance **datacenter-ec2** is now protected from being stopped via the AWS Console, CLI, or API.

---

```bash
aws ec2 describe-instances \
  --region us-east-1 \
  --filters "Name=tag:Name,Values=datacenter-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text
i-0dddbe6a69a39f4d0

aws ec2 modify-instance-attribute \
  --region us-east-1 \
  --instance-id i-0dddbe6a69a39f4d0 \
  --disable-api-stop

aws ec2 describe-instance-attribute \
  --region us-east-1 \
  --instance-id i-0dddbe6a69a39f4d0 \
  --attribute disableApiStop
{
    "InstanceId": "i-0dddbe6a69a39f4d0",
    "DisableApiStop": {
        "Value": true
    }
}
```

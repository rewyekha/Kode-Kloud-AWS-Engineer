# Day 9: Enable Termination Protection for EC2 Instance

As part of the migration, there were some components created under the AWS account. The Nautilus DevOps team created one EC2 instance where they forgot to enable the termination protection which is needed for this instance.

An instance named `nautilus-ec2` already exists in `us-east-1` region. Enable `termination protection` for the same.

Use below given AWS Credentials: (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

| Console URL | [https://000311537719.signin.aws.amazon.com/console?region=us-east-1](https://000311537719.signin.aws.amazon.com/console?region=us-east-1) |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Username    | kk\_labs\_user\_896671                                                                                                                     |
| Password    | 0@@^%dQMdtQy                                                                                                                               |
| Start Time  | Sat Jan 17 03:49:10 UTC 2026                                                                                                               |
| End Time    | Sat Jan 17 04:49:10 UTC 2026                                                                                                               |

\
`Notes:`

* Create the resources only in `us-east-1` region.

```bash
~ on ☁️  (us-east-1) ➜  aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=nautilus-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text
i-0854cd505f18ee159

~ on ☁️  (us-east-1) ➜  aws ec2 modify-instance-attribute \
  --instance-id i-0854cd505f18ee159 \
  --disable-api-termination

~ on ☁️  (us-east-1) ➜  aws ec2 describe-instance-attribute \
  --instance-id i-0854cd505f18ee159 \
  --attribute disableApiTermination
{
    "DisableApiTermination": {
        "Value": true
    },
    "InstanceId": "i-0854cd505f18ee159"
}

~ on ☁️  (us-east-1) ➜  
```

***

## ✅ Day 9: Enable Termination Protection for EC2 Instance

### 🔹 Objective

Enable **termination protection** for an existing EC2 instance named **`nautilus-ec2`** in the **`us-east-1`** region.

Termination protection prevents accidental deletion of critical EC2 instances.

***

### 📌 Important Constraints

* **Region:** `us-east-1` only
* **Instance already exists** (do NOT create a new one)
* **Action required:** Enable termination protection

***

### 🧠 What is Termination Protection?

In **Amazon EC2**, termination protection is a safety feature that blocks the **Terminate** action until protection is disabled.

***

## 🧪 Method 1: AWS CLI (Recommended for KodeKloud)

#### 🔹 Step 1: Login to aws-client host

```bash
ssh aws-client
```

***

#### 🔹 Step 2: Load AWS credentials

```bash
showcreds
```

Copy the **AWS\_ACCESS\_KEY\_ID**, **AWS\_SECRET\_ACCESS\_KEY**, and **AWS\_SESSION\_TOKEN**.

***

#### 🔹 Step 3: Configure AWS CLI

```bash
aws configure
```

Enter:

* **AWS Access Key ID** → from `showcreds`
* **AWS Secret Access Key** → from `showcreds`
* **Default region name** → `us-east-1`
* **Default output format** → `json`

***

#### 🔹 Step 4: Get Instance ID of `nautilus-ec2`

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=nautilus-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text
```

📌 Example output:

```
i-0abc12345def6789
```

***

#### 🔹 Step 5: Enable Termination Protection

```bash
aws ec2 modify-instance-attribute \
  --instance-id i-0abc12345def6789 \
  --disable-api-termination
```

✅ If no output appears, the command was **successful**.

***

#### 🔹 (Optional) Verify Protection Status

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

***

## 🖥️ Method 2: AWS Console (GUI)

#### 🔹 Step 1: Login to Console

Open **AWS Management Console**\
🔗 [https://000311537719.signin.aws.amazon.com/console](https://000311537719.signin.aws.amazon.com/console)

* **Region:** `us-east-1`
* **Username & Password:** (as provided)

***

#### 🔹 Step 2: Navigate to EC2

```
Services → EC2 → Instances
```

***

#### 🔹 Step 3: Select the instance

* Find instance named **`nautilus-ec2`**
* Select the checkbox

***

#### 🔹 Step 4: Enable Termination Protection

```
Actions → Instance settings → Change termination protection
```

* Select ✅ **Enable**
* Click **Save**

<figure><img src=".gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

***

#### 🔹 Step 5: Verify

* Select the instance
* Go to **Details tab**
* Confirm:

```
Termination protection: Enabled
```

***

## 🧾 Final Validation Checklist

✅ Instance name: `nautilus-ec2`\
✅ Region: `us-east-1`\
✅ Termination protection: **Enabled**\
✅ No new resources created

***

### 🏁 KodeKloud Exam Tip

> **Termination protection is NOT enabled by default.**\
> Always use `modify-instance-attribute --disable-api-termination` for CLI questions.


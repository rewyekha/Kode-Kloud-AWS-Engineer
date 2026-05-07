# Day 2 - Security group

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create a security group under default VPC with the following requirements:

* Name of the security group is `xfusion-sg`.
* The description must be `Security group for Nautilus App Servers`
* Add the inbound rule of type `HTTP`, with port range of `80`. Enter the source CIDR range of `0.0.0.0/0`.
* Add another inbound rule of type `SSH`, with port range of `22`. Enter the source CIDR range of `0.0.0.0/0`.

Use below given AWS Credentials: (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

| Console URL | https://<aws-account>.signin.aws.amazon.com/console?region=us-east-1 |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Username    | <REDACTED_USERNAME> |
| Password    | <REDACTED_PASSWORD> |

<figure><img src=".gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

Below are the **AWS CLI steps** to create the required security group in the **default VPC** and add the inbound rules.

>  Assumption:\
> • You are logged into the **aws-client** host\
> • AWS credentials are already configured using `showcreds`\
> • Region is already set (or you can add `--region <region>` if needed)

---

### 1. Verify AWS CLI Configuration

(Optional but recommended)

```bash
aws sts get-caller-identity
```

---

### 2. Get the Default VPC ID

You must create the security group inside the **default VPC**.

```bash
VPC_ID=$(aws ec2 describe-vpcs \
  --filters Name=isDefault,Values=true \
  --query "Vpcs[0].VpcId" \
  --output text)

echo $VPC_ID
```

---

### 3. Create the Security Group

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

---

### 4. Add Inbound Rule for HTTP (Port 80)

Allow HTTP access from anywhere.

```bash
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

---

### 5. Add Inbound Rule for SSH (Port 22)

Allow SSH access from anywhere.

```bash
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

---

### 6. Verify the Security Group

Confirm that the rules were added correctly.

```bash
aws ec2 describe-security-groups \
  --group-ids $SG_ID
```

---

 **Result**

* Security Group Name: `xfusion-sg`
* VPC: Default VPC
* Inbound Rules:
  * HTTP (80) → `0.0.0.0/0`
  * SSH (22) → `0.0.0.0/0`

---

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

# Day 26: Configuring an EC2 Instance as a Web Server with Nginx

This guide demonstrates how to launch a **Ubuntu 20.04 EC2 instance** for the Nautilus project with **Nginx installed**, accessible via HTTP and SSH.

**Region:** `us-east-1`
**Instance Name:** `datacenter-ec2`
**AMI:** `ami-0fb0b230890ccd1e6` (Ubuntu 20.04 LTS)

***

### **Step 0: Set Variables**

```bash
~ on ☁️ (us-east-1) ➜  REGION=us-east-1
~ on ☁️ (us-east-1) ➜  AMI_ID=ami-0fb0b230890ccd1e6
~ on ☁️ (us-east-1) ➜  INSTANCE_NAME=datacenter-ec2
~ on ☁️ (us-east-1) ➜  SG_NAME=datacenter-sg
```

***

### **Step 1: Terminate old instance (if any)**

```bash
~ on ☁️ (us-east-1) ➜  EXISTING_ID=$(aws ec2 describe-instances \
>   --filters "Name=tag:Name,Values=$INSTANCE_NAME" \
>   --query "Reservations[*].Instances[*].InstanceId" --output text --region $REGION)

~ on ☁️ (us-east-1) ➜  if [ -n "$EXISTING_ID" ]; then
>     aws ec2 terminate-instances --instance-ids $EXISTING_ID --region $REGION
>     aws ec2 wait instance-terminated --instance-ids $EXISTING_ID --region $REGION
>     echo "Terminated old instance(s): $EXISTING_ID"
>   fi
{
    "TerminatingInstances": [
        {
            "InstanceId": "i-080df7a0e5a2c9c96",
            "CurrentState": {
                "Code": 48,
                "Name": "terminated"
            },
            "PreviousState": {
                "Code": 48,
                "Name": "terminated"
            }
        },
        {
            "InstanceId": "i-02f787bfa9bb59f3b",
            "CurrentState": {
                "Code": 48,
                "Name": "terminated"
            },
            "PreviousState": {
                "Code": 48,
                "Name": "terminated"
            }
        }
    ]
}
Terminated old instance(s): i-02f787bfa9bb59f3b
i-080df7a0e5a2c9c96
```

***

### **Step 2: Delete old security group (if any)**

```bash
~ on ☁️ (us-east-1) ➜  EXISTING_SG_ID=$(aws ec2 describe-security-groups \
>   --group-names $SG_NAME --query "SecurityGroups[0].GroupId" --output text --region $REGION 2>/dev/null)

~ on ☁️ (us-east-1) ➜  if [ -n "$EXISTING_SG_ID" ]; then
>     aws ec2 delete-security-group --group-id $EXISTING_SG_ID --region $REGION
>     echo "Deleted old security group: $EXISTING_SG_ID"
>   fi
{
    "Return": true,
    "GroupId": "sg-04bc0eba3a165a48d"
}
Deleted old security group: sg-04bc0eba3a165a48d
```

***

### **Step 3: Create new security group (HTTP + SSH)**

```bash
~ on ☁️ (us-east-1) ➜  SG_ID=$(aws ec2 create-security-group \
>   --group-name $SG_NAME \
>   --description "HTTP + SSH access for $INSTANCE_NAME" \
>   --query "GroupId" --output text --region $REGION)

~ on ☁️ (us-east-1) ➜  aws ec2 authorize-security-group-ingress \
>   --group-id $SG_ID --protocol tcp --port 80 --cidr 0.0.0.0/0 --region $REGION

~ on ☁️ (us-east-1) ➜  aws ec2 authorize-security-group-ingress \
>   --group-id $SG_ID --protocol tcp --port 22 --cidr 0.0.0.0/0 --region $REGION

~ on ☁️ (us-east-1) ➜  echo "Created security group: $SG_ID"
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-06294d760d27906cd",
            "GroupId": "sg-01a574ee57f7ad35d",
            "GroupOwnerId": "835363682074",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIpv4": "0.0.0.0/0",
            "SecurityGroupRuleArn": "arn:aws:ec2:us-east-1:835363682074:security-group-rule/sgr-06294d760d27906cd"
        }
    ]
}
Created security group: sg-01a574ee57f7ad35d
```

***

### **Step 4: Create user-data script for Nginx**

```bash
~ on ☁️ (us-east-1) ➜  cat << 'EOF' > user-data.sh
#!/bin/bash
apt-get update -y
apt-get install nginx -y
systemctl enable nginx
systemctl start nginx
EOF
```

***

### **Step 5: Launch Ubuntu EC2 instance with public IP**

```bash
~ on ☁️ (us-east-1) ➜  INSTANCE_ID=$(aws ec2 run-instances \
>   --image-id $AMI_ID \
>   --instance-type t2.micro \
>   --security-group-ids $SG_ID \
>   --user-data file://user-data.sh \
>   --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=$INSTANCE_NAME}]" \
>   --count 1 \
>   --associate-public-ip-address \
>   --query "Instances[0].InstanceId" --output text \
>   --region $REGION)

~ on ☁️ (us-east-1) ➜  echo "Launched new instance: $INSTANCE_ID"
Launched new instance: i-088f7192edad36941
```

***

### **Step 6: Wait until running and get Public IP**

```bash
~ on ☁️ (us-east-1) ➜  aws ec2 wait instance-running --instance-ids $INSTANCE_ID --region $REGION

~ on ☁️ (us-east-1) ➜  PUBLIC_IP=$(aws ec2 describe-instances \
>   --instance-ids $INSTANCE_ID \
>   --query "Reservations[0].Instances[0].PublicIpAddress" --output text \
>   --region $REGION)

~ on ☁️ (us-east-1) ➜  echo "Web server is ready: http://$PUBLIC_IP"
Web server is ready: http://54.91.95.154
```

***

### **Access the Web Server**

* Open browser: `http://54.91.95.154` → should show **Nginx default page**
* SSH access (lab / troubleshooting):

```bash
ssh ubuntu@54.91.95.154 -i your-key.pem
```

> Security group allows port **22** for SSH.

***

#### **Notes**

* Clean, minimal, lab-ready setup.
* Fully automated via **user-data script**.
* Uses **Ubuntu AMI** (required for `apt-get`).
* One-shot setup, no manual configuration.

***

<figure><img src=".gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

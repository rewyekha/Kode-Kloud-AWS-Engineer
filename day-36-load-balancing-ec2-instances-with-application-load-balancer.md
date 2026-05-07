# Day 36: Load Balancing EC2 Instances with Application Load Balancer

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

**Question:**

You are tasked with deploying a web application on AWS using an EC2 instance and an Application Load Balancer (ALB). The workflow involves launching an EC2 instance with Ubuntu 22.04, installing Nginx via user data, configuring security groups, and ensuring the instance is publicly accessible through the ALB.

After performing the initial setup, you notice the following:

1. The EC2 instance is running and Nginx is installed.
2. The instance is registered to the target group of the ALB.
3. The target group reports the instance as `healthy`.
4. Attempting to access the ALB via its DNS name using a browser or `curl` results in a timeout or no response.

Upon investigation:

* The instance resides in `us-east-1f`.
* The ALB is configured for `us-east-1d` and `us-east-1e`.
* Security groups for the ALB and EC2 are configured, with HTTP (port 80) allowed.
* Subnets for the ALB have `MapPublicIpOnLaunch` set to `true`.

You are asked to:

1. Identify the root cause why the ALB is not serving traffic despite the target being healthy.
2. Describe the steps needed to fix the issue, including any subnet, Availability Zone, or instance adjustments.
3. Provide the sequence of AWS CLI commands to relaunch the EC2 instance in the correct subnet, register it with the target group, and verify ALB connectivity.

Include all relevant outputs in your response to demonstrate the solution.

---

## AWS EC2 + ALB Setup with Nginx

This document details the steps to launch an EC2 instance, configure security groups, attach it to an Application Load Balancer (ALB), and verify HTTP access.

---

### 1. Find the Latest Ubuntu AMI

```bash
aws ec2 describe-images \
  --region us-east-1 \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" \
            "Name=state,Values=available" \
  --query "Images | sort_by(@, &CreationDate)[-1].ImageId" \
  --output text
```

**Output:**

```
ami-00de3875b03809ec5
```

---

### 2. Launch EC2 Instance with User Data to Install Nginx

```bash
aws ec2 run-instances \
  --region us-east-1 \
  --image-id ami-00de3875b03809ec5 \
  --instance-type t2.micro \
  --security-group-ids sg-01afb26f5744c0a38 \
  --subnet-id subnet-099e4c2bcb2af3d73 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=devops-ec2}]' \
  --user-data '#!/bin/bash
apt update -y
apt install -y nginx
systemctl start nginx
systemctl enable nginx'
```

**Sample Output:**

```json
{
    "Instances": [
        {
            "InstanceId": "i-081e0cd12d908aa95",
            "State": {"Name": "pending"},
            "SubnetId": "subnet-099e4c2bcb2af3d73",
            "SecurityGroups": [{"GroupId": "sg-01afb26f5744c0a38", "GroupName": "devops-sg"}],
            "PrivateIpAddress": "172.31.51.83"
        }
    ]
}
```

---

### 3. Verify EC2 Instance Status

```bash
aws ec2 describe-instances \
  --region us-east-1 \
  --filters "Name=tag:Name,Values=devops-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text
```

**Output:**

```
i-081e0cd12d908aa95
```

```bash
aws ec2 describe-instances \
  --region us-east-1 \
  --instance-ids i-081e0cd12d908aa95 \
  --query "Reservations[].Instances[].State.Name"
```

**Output:**

```
[
    "running"
]
```

---

### 4. Register EC2 Instance to Target Group

```bash
aws elbv2 register-targets \
  --region us-east-1 \
  --target-group-arn arn:aws:elasticloadbalancing:us-east-1:852719973844:targetgroup/devops-tg/231dcc86ac236825 \
  --targets Id=i-081e0cd12d908aa95
```

```bash
aws elbv2 describe-target-health \
  --region us-east-1 \
  --target-group-arn arn:aws:elasticloadbalancing:us-east-1:852719973844:targetgroup/devops-tg/231dcc86ac236825
```

**Output:**

```json
{
    "TargetHealthDescriptions": [
        {
            "Target": {"Id": "i-081e0cd12d908aa95", "Port": 80},
            "TargetHealth": {"State": "healthy"}
        }
    ]
}
```

---

### 5. Verify ALB Configuration

```bash
aws elbv2 describe-load-balancers \
  --region us-east-1 \
  --names devops-alb \
  --query "LoadBalancers[].DNSName"
```

**Output:**

```
devops-alb-1557128968.us-east-1.elb.amazonaws.com
```

```bash
aws elbv2 describe-load-balancers \
  --region us-east-1 \
  --names devops-alb \
  --query "LoadBalancers[].AvailabilityZones[].SubnetId"
```

**Output:**

```
[
    "subnet-099e4c2bcb2af3d73",
    "subnet-0c24de8c837e3ae2f"
]
```

```bash
aws elbv2 describe-load-balancers \
  --region us-east-1 \
  --names devops-alb \
  --query "LoadBalancers[].State.Code"
```

**Output:**

```
[
    "active"
]
```

---

### 6. Verify Security Group Rules

**ALB Security Group:**

```bash
ALB_SG_ID=sg-015c6a755ee355a68

aws ec2 describe-security-groups \
  --region us-east-1 \
  --group-ids $ALB_SG_ID \
  --query "SecurityGroups[].IpPermissions"
```

**Output:**

```json
[
    [
        {
            "IpProtocol": "-1",
            "UserIdGroupPairs": [{"UserId": "852719973844", "GroupId": "sg-015c6a755ee355a68"}],
            "IpRanges": [],
            "Ipv6Ranges": []
        }
    ]
]
```

**Add HTTP access (TCP 80 from all IPs):**

```bash
aws ec2 authorize-security-group-ingress \
  --region us-east-1 \
  --group-id $ALB_SG_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

**Output:**

```json
{
    "Return": true
}
```

---

### 7. Verify EC2 Public Subnet

```bash
aws ec2 describe-subnets \
  --region us-east-1 \
  --subnet-ids subnet-099e4c2bcb2af3d73 subnet-0c24de8c837e3ae2f \
  --query "Subnets[].MapPublicIpOnLaunch"
```

**Output:**

```
[
    true,
    true
]
```

---

### 8. Verify ALB Access via Browser or cURL

```bash
curl http://devops-alb-1557128968.us-east-1.elb.amazonaws.com
```

**Output:**

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and working.</p>
</body>
</html>
```

---

####  Summary

* EC2 instance deployed with Ubuntu 22.04 and Nginx installed.
* Security groups configured for HTTP traffic.
* Instance registered to ALB target group in the correct Availability Zone.
* ALB is publicly accessible, routing traffic to the EC2 instance successfully.

---

<figure><img src=".gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

---

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

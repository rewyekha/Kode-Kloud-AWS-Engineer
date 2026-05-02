# Day 44: Implementing Auto Scaling for High Availability in AWS

The DevOps team is tasked with setting up a highly available web application using AWS. To achieve this, they plan to use an Auto Scaling Group (ASG) to ensure that the required number of EC2 instances are always running, and an Application Load Balancer (ALB) to distribute traffic across these instances. The goal of this task is to set up an ASG that automatically scales EC2 instances based on **CPU utilization**, and an ALB that directs incoming traffic to the instances. The EC2 instances should have Nginx installed and running to serve web traffic.

1. Create an EC2 launch template named `nautilus-launch-template` that specifies the configuration for the EC2 instances, including the **Amazon Linux 2 AMI**, **t2.micro** instance type, and a security group that allows **HTTP traffic on port 80**.
2. Add a **User Data** script to the launch template to install Nginx on the EC2 instances when they are launched. The script should install Nginx, start the Nginx service, and enable it to start on boot.
3. Create an Auto Scaling Group named `nautilus-asg` that uses the launch template and ensures a minimum of **1 instance**, desired capacity is **1 instance** and a maximum of **2 instances** are running based on **CPU utilization**. Set the target **CPU utilization** to **50%**.
4. Create a target group named `nautilus-tg`, an Application Load Balancer named `nautilus-alb` and configure it to listen on **port 80**. Ensure the ALB is associated with the Auto Scaling Group and distributes traffic across the instances.
5. Configure health checks on the ALB to ensure it routes traffic only to healthy instances.
6. Verify that the ALB's DNS name is accessible and that it displays the default Nginx page served by the EC2 instances.

**Notes:**

* Create the resources only in `us-east-1` region.

## Overview

This document provides a complete, production‑grade GitBook-style documentation of setting up a highly available web application on AWS using Auto Scaling Groups (ASG) and an Application Load Balancer (ALB). The entire setup is executed using the AWS CLI in the `us-east-1` region and validated end‑to‑end.The solution provisions EC2 instances running Nginx, scales them automatically based on CPU utilization, and exposes the application via an internet‑facing ALB.

***

### Architecture Summary

* EC2 Launch Template: Amazon Linux 2, `t2.micro`, Nginx via User Data
* Security Group: Allows inbound HTTP (80)
* Auto Scaling Group: Min 1, Desired 1, Max 2 instances
* Scaling Policy: Target tracking on average CPU utilization (50%)
* Load Balancer: Application Load Balancer (HTTP :80)
* Target Group: Instance targets with health checks on `/`

***

### Prerequisites

* AWS CLI configured with valid credentials
* Default VPC available in `us-east-1`
* IAM permissions for EC2, ELBv2, Auto Scaling, and CloudWatch

***

### Step 1: Identify Default VPC

```shell
VPC_ID=$(aws ec2 describe-vpcs \
  --filters Name=isDefault,Values=true \
  --query 'Vpcs[0].VpcId' \
  --output text)
```

***

### Step 2: Create Security Group

#### Create Security Group

```shell
aws ec2 create-security-group \
  --group-name nautilus-web-sg \
  --description "Allow HTTP traffic" \
  --vpc-id $VPC_ID
```

Output:

```json
{
  "GroupId": "sg-070d8c4d6f3f7ee15",
  "SecurityGroupArn": "arn:aws:ec2:us-east-1:139373540961:security-group/sg-070d8c4d6f3f7ee15"
}
```

#### Allow HTTP Traffic (Port 80)

```shell
SG_ID=$(aws ec2 describe-security-groups \
  --filters Name=group-name,Values=nautilus-web-sg \
  --query 'SecurityGroups[0].GroupId' \
  --output text)

aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

***

### Step 3: Create Launch Template

#### Fetch Latest Amazon Linux 2 AMI

```shell
AMI_ID=$(aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" \
  --query 'Images | sort_by(@,&CreationDate)[-1].ImageId' \
  --output text)
```

#### Create Launch Template with Nginx User Data

```shell
aws ec2 create-launch-template \
  --launch-template-name nautilus-launch-template \
  --version-description "nginx-template" \
  --launch-template-data "{
    \"ImageId\": \"$AMI_ID\",
    \"InstanceType\": \"t2.micro\",
    \"SecurityGroupIds\": [\"$SG_ID\"],
    \"UserData\": \"$(echo '#!/bin/bash
yum update -y
amazon-linux-extras install nginx1 -y
systemctl start nginx
systemctl enable nginx' | base64 -w0)\"
  }"
```

***

### Step 4: Create Target Group

```shell
aws elbv2 create-target-group \
  --name nautilus-tg \
  --protocol HTTP \
  --port 80 \
  --vpc-id $VPC_ID \
  --target-type instance \
  --health-check-path /
```

```shell
TG_ARN=$(aws elbv2 describe-target-groups \
  --names nautilus-tg \
  --query 'TargetGroups[0].TargetGroupArn' \
  --output text)
```

***

### Step 5: Create Application Load Balancer

#### Retrieve Subnets

```shell
SUBNETS=$(aws ec2 describe-subnets \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'Subnets[*].SubnetId' \
  --output text)
```

#### Create ALB

```shell
aws elbv2 create-load-balancer \
  --name nautilus-alb \
  --subnets $SUBNETS \
  --security-groups $SG_ID \
  --scheme internet-facing \
  --type application
```

```shell
ALB_ARN=$(aws elbv2 describe-load-balancers \
  --names nautilus-alb \
  --query 'LoadBalancers[0].LoadBalancerArn' \
  --output text)
```

***

### Step 6: Configure ALB Listener

```shell
aws elbv2 create-listener \
  --load-balancer-arn $ALB_ARN \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=$TG_ARN
```

***

### Step 7: Create Auto Scaling Group

```shell
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name nautilus-asg \
  --launch-template LaunchTemplateName=nautilus-launch-template,Version=1 \
  --min-size 1 \
  --desired-capacity 1 \
  --max-size 2 \
  --vpc-zone-identifier "$(echo $SUBNETS | tr ' ' ',')" \
  --target-group-arns $TG_ARN
```

***

### Step 8: Configure CPU Target Tracking Scaling (50%)

```shell
aws autoscaling put-scaling-policy \
  --policy-name cpu50-policy \
  --auto-scaling-group-name nautilus-asg \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 50.0
  }'
```

***

### Step 9: Validation & Health Checks

#### ASG Instance Verification

```shell
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names nautilus-asg \
  --query 'AutoScalingGroups[0].Instances'
```

#### Target Registration & Health

```shell
aws elbv2 describe-target-health \
  --target-group-arn $TG_ARN
```

Final Healthy State:

```json
"State": "healthy"
```

***

### Step 10: Application Verification

#### Retrieve ALB DNS

```shell
ALB_DNS=$(aws elbv2 describe-load-balancers \
  --names nautilus-alb \
  --query 'LoadBalancers[0].DNSName' \
  --output text)

echo "http://$ALB_DNS"
```

#### Validate via Curl

```shell
curl http://$ALB_DNS
```

Output:

```html
<h1>Welcome to nginx!</h1>
```

***

### Conclusion

This lab demonstrates a fully functional, production‑aligned AWS architecture using:

* Infrastructure created entirely via AWS CLI
* Automatic scaling based on real metrics
* Load-balanced Nginx web servers
* Robust health checks and fault tolerance

* Lab Status: PASSED

***

### Cleanup (Optional)

> Always remember to clean up resources to avoid unnecessary costs.

```shell
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name nautilus-asg --force-delete
aws elbv2 delete-load-balancer --load-balancer-arn $ALB_ARN
aws elbv2 delete-target-group --target-group-arn $TG_ARN
aws ec2 delete-launch-template --launch-template-name nautilus-launch-template
aws ec2 delete-security-group --group-id $SG_ID
```

***

Author: DevOps Engineering TeamPlatform: AWS (us-east-1)Documentation Type: GitBook / Runbook / Lab Evidence

<figure><img src=".gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

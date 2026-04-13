# Day 24: Setting Up an Application Load Balancer for an EC2 Instance

The Nautilus DevOps team is currently working on setting up a simple application on the AWS cloud. They aim to establish an Application Load Balancer (ALB) in front of an EC2 instance where an Nginx server is currently running. While the Nginx server currently serves a sample page, the team plans to deploy the actual application later.

1. Set up an Application Load Balancer named `nautilus-alb`.
2. Create a target group named `nautilus-tg`.
3. Create a security group named `nautilus-sg` to open port `80` for the public.
4. Attach this security group to the ALB.
5. The ALB should route traffic on port `80` to port `80` of the `nautilus-ec2` instance.
6. Make appropriate changes in the default security group attached to the EC2 instance if necessary.

Use below given AWS Credentials: (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

\
`Notes:`

* Create the resources only in `us-east-1` region.
* To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:\
  ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)



***

## AWS Application Load Balancer in Front of EC2 (Nginx)

### Problem Statement

The Nautilus DevOps team is setting up a simple application on AWS. The goal is to configure an **Application Load Balancer (ALB)** in front of an EC2 instance running **Nginx**.

#### Requirements

1. Create an **Application Load Balancer** named `nautilus-alb`.
2. Create a **Target Group** named `nautilus-tg`.
3. Create a **Security Group** named `nautilus-sg` allowing **HTTP (80)** from the internet.
4. Attach the security group to the ALB.
5. Configure ALB to route **HTTP port 80 → EC2 port 80**.
6. Update the **EC2 security group** if required.

Region:

```
us-east-1
```

***

## Architecture

```
Browser
   │
   ▼
Application Load Balancer (nautilus-alb)
   │
   ▼
Target Group (nautilus-tg)
   │
   ▼
EC2 Instance (Nginx :80)
```

***

## GUI Implementation (AWS Console)

### Step 1 — Create Security Group

Navigate:

```
EC2 → Security Groups → Create Security Group
```

Name:

```
nautilus-sg
```

Inbound Rule:

| Type | Port | Source    |
| ---- | ---- | --------- |
| HTTP | 80   | 0.0.0.0/0 |

***

### Step 2 — Create Target Group

Navigate:

```
EC2 → Target Groups → Create Target Group
```

Configuration:

```
Target Type : Instance
Name : nautilus-tg
Protocol : HTTP
Port : 80
VPC : Same as EC2
Health Check Path : /
```

Register target:

```
nautilus-ec2 instance
```

***

### Step 3 — Create Application Load Balancer

Navigate:

```
EC2 → Load Balancers → Create Load Balancer
```

Select:

```
Application Load Balancer
```

Configuration:

```
Name : nautilus-alb
Scheme : Internet-facing
Protocol : HTTP
Port : 80
Security Group : nautilus-sg
```

Listener:

```
HTTP : 80
Forward to : nautilus-tg
```

***

## CLI Verification

### Check Load Balancer DNS

```bash
aws elbv2 describe-load-balancers \
--names nautilus-alb \
--region us-east-1 \
--query 'LoadBalancers[0].DNSName' \
--output text
```

Output:

```
nautilus-alb-488876130.us-east-1.elb.amazonaws.com
```

***

### Check Target Group

```bash
aws elbv2 describe-target-groups \
--names nautilus-tg \
--region us-east-1
```

Output (excerpt):

```json
{
 "TargetGroupName": "nautilus-tg",
 "Protocol": "HTTP",
 "Port": 80,
 "HealthCheckPath": "/"
}
```

***

### Verify Target Health

```bash
aws elbv2 describe-target-health \
--target-group-arn $(aws elbv2 describe-target-groups \
--names nautilus-tg \
--region us-east-1 \
--query 'TargetGroups[0].TargetGroupArn' \
--output text)
```

Output:

```json
{
 "TargetHealthDescriptions": [
  {
   "Target": {
    "Id": "i-0ef2ad84250aabd0b",
    "Port": 80
   },
   "TargetHealth": {
    "State": "healthy"
   }
  }
 ]
}
```

Target is **healthy**.

***

### Verify Listener

```bash
aws elbv2 describe-listeners \
--load-balancer-arn $(aws elbv2 describe-load-balancers \
--names nautilus-alb \
--region us-east-1 \
--query 'LoadBalancers[0].LoadBalancerArn' \
--output text)
```

Output:

```json
{
 "Port": 80,
 "Protocol": "HTTP",
 "DefaultActions": [
  {
   "Type": "forward",
   "TargetGroupArn": "nautilus-tg"
  }
 ]
}
```

Listener correctly forwards traffic to target group.

***

## Initial Test (FAILED)

Testing using curl:

```bash
curl http://nautilus-alb-488876130.us-east-1.elb.amazonaws.com
```

Result:

```
(no response / timeout)
```

Browser error:

```
The connection has timed out
```

***

## Debugging Process

### Check ALB Security Group

```bash
aws elbv2 describe-load-balancers \
--names nautilus-alb \
--region us-east-1 \
--query 'LoadBalancers[0].SecurityGroups' \
--output text
```

Output:

```
sg-034ffebee3f8e8793
```

***

### Inspect Security Group Rules

```bash
aws ec2 describe-security-groups \
--group-ids sg-034ffebee3f8e8793 \
--region us-east-1 \
--query 'SecurityGroups[0].IpPermissions'
```

Output:

```json
[
 {
  "IpProtocol": "-1",
  "UserIdGroupPairs": [
   {
    "GroupId": "sg-034ffebee3f8e8793"
   }
  ]
 }
]
```

#### Problem

The security group **only allowed traffic from itself**.

```
Source: sg-034ffebee3f8e8793
```

It did **NOT allow internet traffic**.

Therefore:

```
Internet → ALB ❌ blocked
ALB → EC2 ✅ working
```

Health checks passed because they originate **inside the VPC**.

***

## Fixing the Issue

Add inbound HTTP rule:

```bash
aws ec2 authorize-security-group-ingress \
--group-id sg-034ffebee3f8e8793 \
--protocol tcp \
--port 80 \
--cidr 0.0.0.0/0 \
--region us-east-1
```

Response:

```json
{
 "Return": true
}
```

***

### Verify Updated Rule

```bash
aws ec2 describe-security-groups \
--group-ids sg-034ffebee3f8e8793 \
--query 'SecurityGroups[0].IpPermissions'
```

Output:

```json
[
 {
  "IpProtocol": "tcp",
  "FromPort": 80,
  "ToPort": 80,
  "IpRanges": [
   {
    "CidrIp": "0.0.0.0/0"
   }
  ]
 }
]
```

Now HTTP is publicly accessible.

***

## Final Validation

Test again:

```bash
curl http://nautilus-alb-488876130.us-east-1.elb.amazonaws.com
```

Output:

```html
<h1>Welcome to nginx!</h1>
```

Nginx page successfully served through ALB.

***

## Final Result

Working flow:

```
Browser
   │
   ▼
ALB (nautilus-alb :80)
   │
   ▼
Target Group (nautilus-tg)
   │
   ▼
EC2 Instance (Nginx :80)
```

***

## Key Lessons Learned

#### 1️⃣ Health checks do not guarantee internet accessibility

ALB health checks originate from **inside the VPC**, so they can succeed even when public access is blocked.

***

#### 2️⃣ Always verify ALB security groups

A common mistake is missing:

```
HTTP 80 → 0.0.0.0/0
```

***

#### 3️⃣ Debugging checklist for ALB

Check in this order:

1. ALB DNS
2. Listener
3. Target group
4. Target health
5. ALB security group
6. EC2 security group



<figure><img src=".gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>


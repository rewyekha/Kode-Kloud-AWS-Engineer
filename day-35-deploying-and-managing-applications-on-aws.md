# Day 35: Deploying and Managing Applications on AWS

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

## AWS: Connect EC2 to Private RDS MySQL Instance with PHP Application

> **Platform:** KodeKloud | **Cloud:** AWS | **Region:** us-east-1 **Difficulty:** Intermediate | **Topic:** AWS RDS, EC2, MySQL, Security Groups, SSH, PHP, Apache

---

### Table of Contents

1. [Lab Overview](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-overview)
2. [Lab Objectives](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-objectives)
3. [Prerequisites](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#prerequisites)
4. [Architecture](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#architecture)
5. [Phase 1: Set Environment Variables](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-1-set-environment-variables)
6. [Phase 2: Gather VPC and EC2 Information](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-2-gather-vpc-and-ec2-information)
7. [Phase 3: Create DB Subnet Group and Security Groups](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-3-create-db-subnet-group-and-security-groups)
8. [Phase 4: Create the RDS Instance](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-4-create-the-rds-instance)
9. [Phase 5: Generate SSH Key and Add to EC2](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-5-generate-ssh-key-and-add-to-ec2)
10. [Phase 6: Configure and Deploy the PHP File](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-6-configure-and-deploy-the-php-file)
11. [Phase 7: Verify the Application](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-7-verify-the-application)
12. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
13. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)
14. [Resource Reference](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#resource-reference)

---

### Lab Overview

The Nautilus DevOps team needs a new private RDS instance for their application. They need to set up a MySQL database and ensure that their existing EC2 instance can connect to it. A PHP application running on Apache on the EC2 instance connects to the RDS and confirms the connection via a browser-accessible page.

---

### Lab Objectives

1. Create a private RDS instance named `devops-rds` using sandbox template, `MySQL v8.4.5`, `db.t3.micro`, `gp2`, `5GiB` storage.
2. Master username `devops_admin`, database name `devops_db`.
3. Configure security groups — allow `devops-ec2` to connect to RDS on port `3306` and open port `80` on EC2.
4. Create SSH key on `aws-client` host and add public key to root user on EC2 for passwordless access.
5. Copy `index.php` from `/root` on aws-client to `/var/www/html/` on `devops-ec2` with RDS connection details.
6. Verify `Connected successfully` message in browser via EC2 public IP.

---

### Prerequisites

| Field         | Value                                                                 |
| ------------- | --------------------------------------------------------------------- |
| Console URL   | https://<aws-account>.signin.aws.amazon.com/console?region=us-east-1 |
| Username      | <REDACTED_USERNAME> |
| Password      | <REDACTED_PASSWORD> |
| Region        | `us-east-1`                                                           |
| Access Method | AWS CLI on `aws-client` host                                          |

---

### Architecture

```bash
┌─────────────────────────────────────────────────────────────────────┐
│                        AWS us-east-1                                │
│                                                                     │
│   ┌──────────────────────────────────────────────────────────────┐  │
│   │                  Default VPC                                 │  │
│   │                                                              │  │
│   │  ┌─────────────────────┐      ┌───────────────────────────┐  │  │
│   │  │    devops-ec2        │      │      devops-rds          │  │  │
│   │  │    (Apache + PHP)    │─────▶│      MySQL 8.4.5         │  │  │
│   │  │    Port 80 open      │ 3306 │      db.t3.micro         │  │  │
│   │  │    Port 22 open      │      │      gp2 / 5GiB          │  │  │
│   │  │    SG: EC2-SG        │      │      Private (no public) │  │  │
│   │  └──────────┬───────────┘      │      SG: RDS-SG           │ │  │
│   │             │                  └───────────────────────────┘ │  │
│   │             │ curl                                           │  │
│   │             ▼                                                │  │
│   │    Browser: Connected successfully                           │  │
│   └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│   aws-client host                                                   │
│   /root/index.php → scp → /var/www/html/index.php                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

### Phase 1: Set Environment Variables

```bash
RDS_ID="devops-rds"
DB_ENGINE="mysql"
DB_ENGINE_VERSION="8.4.5"
DB_INSTANCE_CLASS="db.t3.micro"
DB_NAME="devops_db"
MASTER_USER="devops_admin"
MASTER_PASS="AdminPa55"
STORAGE_TYPE="gp2"
STORAGE_SIZE=5
REGION="us-east-1"
```

**Terminal Output:**

```bash
~ via  on   (us-east-1)   RDS_ID="devops-rds"
DB_ENGINE="mysql"
DB_ENGINE_VERSION="8.4.5"
DB_INSTANCE_CLASS="db.t3.micro"
DB_NAME="devops_db"
MASTER_USER="devops_admin"
MASTER_PASS="AdminPa55"
STORAGE_TYPE="gp2"
STORAGE_SIZE=5
REGION="us-east-1"

~ via  on   (us-east-1)
```

---

### Phase 2: Gather VPC and EC2 Information

#### Get Default VPC ID

```bash
VPC_ID=$(aws ec2 describe-vpcs \
  --filters "Name=isDefault,Values=true" \
  --query "Vpcs[0].VpcId" --output text --region $REGION)
echo $VPC_ID
```

**Terminal Output:**

```
vpc-0301d8e20b66b7589
```

#### Get All Subnets in VPC

```bash
SUBNET_IDS=$(aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=$VPC_ID" \
  --query "Subnets[*].SubnetId" --output text --region $REGION)
echo $SUBNET_IDS
```

**Terminal Output:**

```
subnet-00e938a8ab3f61a17 subnet-0358aacb2d38f4bdf subnet-06860512823335349 subnet-0d72f2bcd36be0aae subnet-0d546167c46a1645c subnet-0fe249b33c579da90
```

#### Get EC2 Instance Details

```bash
EC2_INSTANCE_ID=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=devops-ec2" \
  --query "Reservations[0].Instances[0].InstanceId" \
  --output text --region $REGION)
echo $EC2_INSTANCE_ID

EC2_PUBLIC_IP=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=devops-ec2" \
  --query "Reservations[0].Instances[0].PublicIpAddress" \
  --output text --region $REGION)
echo $EC2_PUBLIC_IP

EC2_SG=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=devops-ec2" \
  --query "Reservations[0].Instances[0].SecurityGroups[0].GroupId" \
  --output text --region $REGION)
echo $EC2_SG
```

**Terminal Output:**

```
i-00f5d164f5ab1abb7
54.234.87.25
sg-0ae121a0995aef738
```

---

### Phase 3: Create DB Subnet Group and Security Groups

#### Create DB Subnet Group

```bash
aws rds create-db-subnet-group \
  --db-subnet-group-name devops-subnet-group \
  --db-subnet-group-description "Subnet group for devops-rds" \
  --subnet-ids $SUBNET_IDS \
  --region $REGION
```

**Terminal Output:**

```bash
{
    "DBSubnetGroup": {
        "DBSubnetGroupName": "devops-subnet-group",
        "DBSubnetGroupDescription": "Subnet group for devops-rds",
        "VpcId": "vpc-0301d8e20b66b7589",
        "SubnetGroupStatus": "Complete",
        "Subnets": [
            {
                "SubnetIdentifier": "subnet-00e938a8ab3f61a17",
                "SubnetAvailabilityZone": { "Name": "us-east-1d" },
                "SubnetStatus": "Active"
            },
            {
                "SubnetIdentifier": "subnet-0358aacb2d38f4bdf",
                "SubnetAvailabilityZone": { "Name": "us-east-1f" },
                "SubnetStatus": "Active"
            },
            {
                "SubnetIdentifier": "subnet-06860512823335349",
                "SubnetAvailabilityZone": { "Name": "us-east-1b" },
                "SubnetStatus": "Active"
            },
            {
                "SubnetIdentifier": "subnet-0d546167c46a1645c",
                "SubnetAvailabilityZone": { "Name": "us-east-1e" },
                "SubnetStatus": "Active"
            },
            {
                "SubnetIdentifier": "subnet-0d72f2bcd36be0aae",
                "SubnetAvailabilityZone": { "Name": "us-east-1c" },
                "SubnetStatus": "Active"
            },
            {
                "SubnetIdentifier": "subnet-0fe249b33c579da90",
                "SubnetAvailabilityZone": { "Name": "us-east-1a" },
                "SubnetStatus": "Active"
            }
        ],
        "DBSubnetGroupArn": "arn:aws:rds:us-east-1:417003076066:subgrp:devops-subnet-group"
    }
}
```

#### Create RDS Security Group

```bash
RDS_SG=$(aws ec2 create-security-group \
  --group-name devops-rds-sg \
  --description "RDS security group" \
  --vpc-id $VPC_ID \
  --query "GroupId" --output text --region $REGION)
echo $RDS_SG
```

**Terminal Output:**

```
sg-0288d30aaa41a94e7
```

#### Allow EC2 to Connect to RDS on Port 3306

```bash
aws ec2 authorize-security-group-ingress \
  --group-id $RDS_SG \
  --protocol tcp \
  --port 3306 \
  --source-group $EC2_SG \
  --region $REGION
```

**Terminal Output:**

```bash
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-041f4d7c96ce05343",
            "GroupId": "sg-0288d30aaa41a94e7",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 3306,
            "ToPort": 3306,
            "ReferencedGroupInfo": {
                "GroupId": "sg-0ae121a0995aef738"
            }
        }
    ]
}
```

#### Open Port 80 on EC2 Security Group

```bash
aws ec2 authorize-security-group-ingress \
  --group-id $EC2_SG \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0 \
  --region $REGION
```

**Terminal Output:**

```bash
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-05ae8ac5869c91d88",
            "GroupId": "sg-0ae121a0995aef738",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
```

#### Open Port 22 on EC2 Security Group

```bash
aws ec2 authorize-security-group-ingress \
  --group-id $EC2_SG \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0 \
  --region $REGION
```

**Terminal Output:**

```bash
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-09b937eed71e4e878",
            "GroupId": "sg-0ae121a0995aef738",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
```

---

### Phase 4: Create the RDS Instance

```bash
aws rds create-db-instance \
  --db-instance-identifier $RDS_ID \
  --db-instance-class $DB_INSTANCE_CLASS \
  --engine $DB_ENGINE \
  --engine-version $DB_ENGINE_VERSION \
  --master-username $MASTER_USER \
  --master-user-password <REDACTED_PASSWORD> \
  --db-name $DB_NAME \
  --storage-type $STORAGE_TYPE \
  --allocated-storage $STORAGE_SIZE \
  --no-publicly-accessible \
  --db-subnet-group-name devops-subnet-group \
  --vpc-security-group-ids $RDS_SG \
  --backup-retention-period 1 \
  --no-multi-az \
  --region $REGION
```

**Terminal Output (key fields):**

```bash
{
    "DBInstance": {
        "DBInstanceIdentifier": "devops-rds",
        "DBInstanceClass": "db.t3.micro",
        "Engine": "mysql",
        "DBInstanceStatus": "creating",
        "MasterUsername": "devops_admin",
        "DBName": "devops_db",
        "AllocatedStorage": 5,
        "StorageType": "gp2",
        "EngineVersion": "8.4.5",
        "PubliclyAccessible": false,
        "MultiAZ": false,
        "DBSubnetGroup": {
            "DBSubnetGroupName": "devops-subnet-group",
            "VpcId": "vpc-0301d8e20b66b7589"
        },
        "DBInstanceArn": "arn:aws:rds:us-east-1:417003076066:db:devops-rds"
    }
}
```

#### Wait for Instance to Become Available

```bash
aws rds wait db-instance-available \
  --db-instance-identifier $RDS_ID \
  --region $REGION
```

> This command blocks and returns silently when the instance reaches `available` status. It takes approximately 5–10 minutes.

#### Get RDS Endpoint

```bash
RDS_ENDPOINT=$(aws rds describe-db-instances \
  --db-instance-identifier $RDS_ID \
  --query "DBInstances[0].Endpoint.Address" \
  --output text --region $REGION)
echo $RDS_ENDPOINT
```

**Terminal Output:**

```
devops-rds.c3yymq8o4wjf.us-east-1.rds.amazonaws.com
```

---

### Phase 5: Generate SSH Key and Add to EC2

#### Generate SSH Key on aws-client

```bash
if [ ! -f /root/.ssh/id_rsa ]; then
  ssh-keygen -t rsa -b 2048 -f /root/.ssh/id_rsa -N ""
fi
cat /root/.ssh/id_rsa.pub
```

**Terminal Output:**

```bash
Generating public/private rsa key pair.
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:YydTeyDxWpJ3SVCEdtUON7zK7gDxWQK/pE+REk8pCTE root@aws-client
The key's randomart image is:
+---[RSA 2048]----+
|       E+oo*=.o. |
|        .*Ooo..oo|
|        ++BOo. +o|
|         B*+*  ..|
|        Soo=o .  |
|       . =+. o   |
|           o.    |
|            ..   |
|            ..   |
+----[SHA256]-----+
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCcsejUmPIbE4LMEx2ghG4yJt70G09vmov3sagAhWG08OGhx1k/3YkumXLbHN6pBJv5mhcKCdYmzmediU6Ig7z2ytg7c42rD8sVKsQr+HnMqz7tXw6kkfyttyj2lKIAOV7N10dfFKOTFEl7yn4JqIzEMCUC2Pdmq6dInivZ2QWy2BOlgvq2g+XjVrmOg0w6MpoNpClpeU1enLRncNpCth35zQkbjGYIlk8sErQbsZWd4BVHi9A8OUPZTU3SQbnnn5/w9sSX+jkOWqz3N7OULhVxlXHN9OWgy84Cy9vyQBn5jG2x/9VLdUSRCQGVLkEJINevBOqFmwpX3L4YKi45r1hr root@aws-client
```

#### Add Public Key to EC2 via AWS Console

The SSH public key was added to the EC2 instance using **EC2 Instance Connect** from the AWS Console:

1. Go to AWS Console → **EC2** → **Instances** → select `devops-ec2`
2. Click **Connect** → **EC2 Instance Connect** tab → click **Connect**
3. Inside the browser terminal, switch to root:

```bash
sudo su -
mkdir -p /root/.ssh
chmod 700 /root/.ssh
echo "ssh-rsa AAAA... root@aws-client" >> /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys
```

After the public key was added, passwordless SSH from aws-client to EC2 as root was confirmed working.

---

### Phase 6: Configure and Deploy the PHP File

#### View Original index.php

```bash
cat /root/index.php
```

**Terminal Output:**

```bash
<?php
$dbname = '<dbname>';
$dbuser = '<dbuser>';
$dbpass = '<dbpass>';
$dbhost = '<dbhost>';

$link = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'");
mysqli_select_db($link, $dbname) or die("Could not open the db '$dbname'");

$test_query = "SHOW TABLES FROM $dbname";
$result = mysqli_query($link, $test_query);

$tblCnt = 0;
while($tbl = mysqli_fetch_array($result)) {
  $tblCnt++;
}

if (!$tblCnt) {
  echo "Connected successfully<br />\n";
} else {
  echo "Connected successfully<br />\n";
}
?>
```

#### Replace Placeholders with Real Values

```bash
sed -i "s|<dbhost>|$RDS_ENDPOINT|g" /root/index.php
sed -i "s|<dbuser>|$MASTER_USER|g" /root/index.php
sed -i "s|<dbpass>|$MASTER_PASS|g" /root/index.php
sed -i "s|<dbname>|$DB_NAME|g" /root/index.php
cat /root/index.php
```

**Terminal Output:**

```bash
<?php
$dbname = 'devops_db';
$dbuser = 'devops_admin';
$dbpass = 'AdminPa55';
$dbhost = 'devops-rds.c3yymq8o4wjf.us-east-1.rds.amazonaws.com';

$link = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'");
mysqli_select_db($link, $dbname) or die("Could not open the db '$dbname'");

$test_query = "SHOW TABLES FROM $dbname";
$result = mysqli_query($link, $test_query);

$tblCnt = 0;
while($tbl = mysqli_fetch_array($result)) {
  $tblCnt++;
}

if (!$tblCnt) {
  echo "Connected successfully<br />\n";
} else {
  echo "Connected successfully<br />\n";
}
?>
```

All four placeholders replaced with actual values.

#### Copy PHP File to EC2

```bash
scp -i /root/.ssh/id_rsa \
  -o StrictHostKeyChecking=no \
  /root/index.php root@$EC2_PUBLIC_IP:/var/www/html/index.php
```

#### Remove Default index.html

```bash
ssh -i /root/.ssh/id_rsa \
  -o StrictHostKeyChecking=no \
  root@$EC2_PUBLIC_IP \
  "rm -f /var/www/html/index.html"
```

---

### Phase 7: Verify the Application

```bash
curl http://$EC2_PUBLIC_IP/index.php
```

**Terminal Output:**

```
Connected successfully
```

The PHP application successfully connected to the private RDS MySQL instance and returned the expected response.

---

### Lab Complete

| Requirement                 | Detail                                | Status    |
| --------------------------- | ------------------------------------- | --------- |
| RDS instance name           | `devops-rds`                          | Confirmed |
| Engine                      | `MySQL 8.4.5`                         | Confirmed |
| Instance class              | `db.t3.micro`                         | Confirmed |
| Storage type                | `gp2`                                 | Confirmed |
| Storage size                | `5GiB`                                | Confirmed |
| Database name               | `devops_db`                           | Confirmed |
| Master username             | `devops_admin`                        | Confirmed |
| Publicly accessible         | `false` (private)                     | Confirmed |
| RDS status                  | `available`                           | Confirmed |
| RDS SG — port 3306 from EC2 | Inbound rule added                    | Confirmed |
| EC2 SG — port 80 open       | Inbound rule added                    | Confirmed |
| EC2 SG — port 22 open       | Inbound rule added                    | Confirmed |
| SSH key on aws-client       | `/root/.ssh/id_rsa` generated         | Confirmed |
| Public key on EC2 root      | Added to `/root/.ssh/authorized_keys` | Confirmed |
| `index.php` deployed        | `/var/www/html/index.php`             | Confirmed |
| Application response        | `Connected successfully`              | Confirmed |

---

### Key Concepts

#### Why the EC2 Must Use Console to Add the SSH Key

AWS EC2 Instance Connect provides a temporary 60-second window for SSH access using a pushed public key. In this lab, the EC2 instance had no existing SSH key configured for the root user. The only reliable method to add the key was to connect via the AWS Console browser terminal, switch to root, and manually append the public key to `/root/.ssh/authorized_keys`. Once added, the key persists permanently and allows direct SSH from the aws-client.

#### Source Group vs CIDR in Security Group Rules

The RDS inbound rule used `--source-group $EC2_SG` instead of a CIDR range:

```bash
# Source group approach — only EC2 instances in EC2_SG can connect
aws ec2 authorize-security-group-ingress \
  --group-id $RDS_SG \
  --protocol tcp \
  --port 3306 \
  --source-group $EC2_SG

# CIDR approach — any IP in that range can connect (less secure)
aws ec2 authorize-security-group-ingress \
  --group-id $RDS_SG \
  --protocol tcp \
  --port 3306 \
  --cidr 10.0.0.0/16
```

Using a source security group is more secure because access is granted based on group membership rather than IP address — only instances belonging to `EC2_SG` can reach the RDS on port 3306.

#### `sed -i` for In-Place File Substitution

```bash
sed -i "s|<dbhost>|$RDS_ENDPOINT|g" /root/index.php
```

The `s|old|new|g` pattern replaces all occurrences of `old` with `new`. The `|` delimiter was used instead of the standard `/` because the RDS endpoint contains dots which would need escaping with `/` as a delimiter. Using `|` avoids this complexity entirely.

#### Why index.html Must Be Removed

Apache serves `index.html` before `index.php` by default as defined in `/etc/apache2/mods-enabled/dir.conf`. If both files exist in `/var/www/html/`, the browser receives the HTML file and never executes the PHP. Removing `index.html` ensures Apache falls through to `index.php` and executes the database connection code.

#### Private RDS — No Public Endpoint

The `--no-publicly-accessible` flag means the RDS instance has no public DNS entry and cannot be reached from the internet. Only resources within the same VPC that have a matching security group rule can establish a connection. This is the correct configuration for production database instances.

---

### Resource Reference

| Resource           | Type                   | Value                                                 |
| ------------------ | ---------------------- | ----------------------------------------------------- |
| VPC                | Default VPC            | `vpc-0301d8e20b66b7589`                               |
| Subnets            | 6 subnets across 6 AZs | `subnet-00e938a8ab3f61a17` ...                        |
| DB Subnet Group    | RDS                    | `devops-subnet-group`                                 |
| RDS Security Group | EC2 SG                 | `sg-0288d30aaa41a94e7`                                |
| EC2 Security Group | EC2 SG                 | `sg-0ae121a0995aef738`                                |
| EC2 Instance ID    | EC2                    | `i-00f5d164f5ab1abb7`                                 |
| EC2 Public IP      | Network                | `54.234.87.25`                                        |
| RDS Instance       | MySQL                  | `devops-rds`                                          |
| RDS Endpoint       | DNS                    | `devops-rds.c3yymq8o4wjf.us-east-1.rds.amazonaws.com` |
| RDS ARN            | AWS ARN                | `arn:aws:rds:us-east-1:417003076066:db:devops-rds`    |
| Database           | MySQL DB               | `devops_db`                                           |
| Master User        | MySQL                  | `devops_admin`                                        |
| PHP File           | Apache                 | `/var/www/html/index.php`                             |

---

_Lab completed on 2026-04-01 | AWS Region: us-east-1 | Platform: KodeKloudbas_

<figure><img src=".gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

---

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

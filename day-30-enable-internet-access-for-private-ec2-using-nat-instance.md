# Day 30: Enable Internet Access for Private EC2 using NAT Instance

## NAT Instance Lab: Enabling Internet Access for a Private EC2 Instance

> **Platform:** KodeKloud | **Cloud:** AWS | **Region:** us-east-1
>
> **Difficulty:** Intermediate | **Topic:** Networking, NAT, VPC, EC2, S3

***

1. [Lab Overview](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-overview)
2. [Architecture Diagram](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#architecture-diagram)
3. [Prerequisites & Existing Resources](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#prerequisites--existing-resources)
4. [Lab Objectives](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-objectives)
5. [Phase 1: Gather Existing Resource Information](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-1-gather-existing-resource-information)
6. [Phase 2: Create Public Subnet](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-2-create-public-subnet)
7. [Phase 3: Internet Gateway Setup](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-3-internet-gateway-setup)
8. [Phase 4: Security Group for NAT Instance](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-4-security-group-for-nat-instance)
9. [Phase 5: Launch the NAT Instance](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-5-launch-the-nat-instance)
10. [Phase 6: Update Private Subnet Route Table](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-6-update-private-subnet-route-table)
11. [Phase 7: Verify the Setup](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#phase-7-verify-the-setup)
12. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)
13. [Resource Summary](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#resource-summary)

***

### Lab Overview

The Nautilus DevOps team needs to enable internet access for an EC2 instance running in a **private subnet**, so it can upload a test file to a public S3 bucket. To minimize costs, the team has chosen to use a **NAT Instance** instead of a NAT Gateway.

> ⚠️ **Note:** `iptables` is not installed by default on Amazon Linux 2023. It must be installed and enabled before configuring NAT.

***

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                  datacenter-priv-vpc                    │
│                   (10.1.0.0/16)                         │
│                                                         │
│  ┌──────────────────────┐  ┌───────────────────────┐   │
│  │  datacenter-priv-    │  │  datacenter-pub-       │   │
│  │  subnet              │  │  subnet                │   │
│  │  (10.1.1.0/24)       │  │  (10.1.2.0/24)         │   │
│  │                      │  │                        │   │
│  │  ┌────────────────┐  │  │  ┌─────────────────┐  │   │
│  │  │datacenter-priv │  │  │  │ datacenter-nat- │  │   │
│  │  │    -ec2        │──┼──┼─▶│   instance      │  │   │
│  │  │(cron → S3 job) │  │  │  │  (NAT + iptables│  │   │
│  │  └────────────────┘  │  │  │   MASQUERADE)   │  │   │
│  │                      │  │  └────────┬────────┘  │   │
│  └──────────────────────┘  └───────────┼───────────┘   │
│                                        │               │
└────────────────────────────────────────┼───────────────┘
                                         ▼
                               ┌──────────────────┐
                               │  datacenter-igw  │
                               │ (Internet Gateway│
                               └────────┬─────────┘
                                        │
                                        ▼
                               ┌──────────────────┐
                               │    Internet      │
                               └────────┬─────────┘
                                        │
                                        ▼
                               ┌──────────────────┐
                               │  S3 Bucket:      │
                               │ datacenter-nat-  │
                               │    17811         │
                               └──────────────────┘
```

***

### Prerequisites & Existing Resources

The following resources already exist in the environment:

| Resource       | Name                     | Details                                    |
| -------------- | ------------------------ | ------------------------------------------ |
| VPC            | `datacenter-priv-vpc`    | CIDR: `10.1.0.0/16`                        |
| Private Subnet | `datacenter-priv-subnet` | CIDR: `10.1.1.0/24`, AZ: `us-east-1a`      |
| Private EC2    | `datacenter-priv-ec2`    | Running in private subnet                  |
| S3 Bucket      | `datacenter-nat-17811`   | Public bucket for test file upload         |
| Cron Job       | on `datacenter-priv-ec2` | Uploads `datacenter-test.txt` every minute |

***

### Lab Objectives

* ✅ Create a new **public subnet** named `datacenter-pub-subnet` in the existing VPC
* ✅ Launch a **NAT Instance** named `datacenter-nat-instance` in the public subnet using Amazon Linux 2023
* ✅ Configure the NAT instance with `iptables` MASQUERADE rules
* ✅ Route private subnet traffic through the NAT instance
* ✅ Verify `datacenter-test.txt` appears in the S3 bucket

***

### Phase 1: Gather Existing Resource Information

Before creating anything, collect all existing resource IDs.

#### Get VPC ID

```bash
aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=datacenter-priv-vpc" \
  --query "Vpcs[0].VpcId" \
  --output text \
  --region us-east-1
```

**Output:**

```
vpc-020c460f013c6d7a2
```

#### Get VPC CIDR Block

```bash
aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=datacenter-priv-vpc" \
  --query "Vpcs[0].CidrBlock" \
  --output text \
  --region us-east-1
```

**Output:**

```
10.1.0.0/16
```

#### Get Private Subnet Details

```bash
aws ec2 describe-subnets \
  --filters "Name=tag:Name,Values=datacenter-priv-subnet" \
  --query "Subnets[0].{SubnetId:SubnetId,AZ:AvailabilityZone,CIDR:CidrBlock}" \
  --output table \
  --region us-east-1
```

**Output:**

```
-----------------------------------------------------------
|                     DescribeSubnets                     |
+------------+---------------+----------------------------+
|     AZ     |     CIDR      |         SubnetId           |
+------------+---------------+----------------------------+
|  us-east-1a|  10.1.1.0/24  |  subnet-0010f478711a26440  |
+------------+---------------+----------------------------+
```

#### Get Private Subnet Route Table

```bash
aws ec2 describe-route-tables \
  --filters "Name=association.subnet-id,Values=subnet-0010f478711a26440" \
  --query "RouteTables[0].RouteTableId" \
  --output text \
  --region us-east-1
```

**Output:**

```
rtb-0e9bf061b3aa4ca15
```

#### Collected Values

| Resource               | ID / Value                 |
| ---------------------- | -------------------------- |
| VPC ID                 | `vpc-020c460f013c6d7a2`    |
| VPC CIDR               | `10.1.0.0/16`              |
| Private Subnet ID      | `subnet-0010f478711a26440` |
| Private Subnet CIDR    | `10.1.1.0/24`              |
| Availability Zone      | `us-east-1a`               |
| Private Route Table ID | `rtb-0e9bf061b3aa4ca15`    |

***

### Phase 2: Create Public Subnet

Create a public subnet in the same AZ as the private subnet using a non-overlapping CIDR.

#### Create the Subnet

```bash
aws ec2 create-subnet \
  --vpc-id vpc-020c460f013c6d7a2 \
  --cidr-block 10.1.2.0/24 \
  --availability-zone us-east-1a \
  --region us-east-1 \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=datacenter-pub-subnet}]'
```

**Output:**

```json
{
    "Subnet": {
        "AvailabilityZoneId": "use1-az2",
        "MapCustomerOwnedIpOnLaunch": false,
        "OwnerId": "495779504297",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "Tags": [
            {
                "Key": "Name",
                "Value": "datacenter-pub-subnet"
            }
        ],
        "SubnetArn": "arn:aws:ec2:us-east-1:495779504297:subnet/subnet-0a8ea30f1103e6408",
        "EnableDns64": false,
        "Ipv6Native": false,
        "SubnetId": "subnet-0a8ea30f1103e6408",
        "State": "available",
        "VpcId": "vpc-020c460f013c6d7a2",
        "CidrBlock": "10.1.2.0/24",
        "AvailableIpAddressCount": 251,
        "AvailabilityZone": "us-east-1a",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false
    }
}
```

**Public Subnet ID:** `subnet-0a8ea30f1103e6408`

#### Enable Auto-Assign Public IP

```bash
aws ec2 modify-subnet-attribute \
  --subnet-id subnet-0a8ea30f1103e6408 \
  --map-public-ip-on-launch \
  --region us-east-1
```

_(No output — success is silent)_

***

### Phase 3: Internet Gateway Setup

#### Create Internet Gateway

```bash
aws ec2 create-internet-gateway \
  --region us-east-1 \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=datacenter-igw}]'
```

**Output:**

```json
{
    "InternetGateway": {
        "Attachments": [],
        "InternetGatewayId": "igw-03c237657533926b7",
        "OwnerId": "495779504297",
        "Tags": [
            {
                "Key": "Name",
                "Value": "datacenter-igw"
            }
        ]
    }
}
```

**IGW ID:** `igw-03c237657533926b7`

#### Attach IGW to VPC

```bash
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-03c237657533926b7 \
  --vpc-id vpc-020c460f013c6d7a2 \
  --region us-east-1
```

_(No output — success is silent)_

#### Create Public Route Table

```bash
aws ec2 create-route-table \
  --vpc-id vpc-020c460f013c6d7a2 \
  --region us-east-1 \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=datacenter-pub-rt}]'
```

**Output:**

```json
{
    "RouteTable": {
        "Associations": [],
        "PropagatingVgws": [],
        "RouteTableId": "rtb-00410046bcf546ea9",
        "Routes": [
            {
                "DestinationCidrBlock": "10.1.0.0/16",
                "GatewayId": "local",
                "Origin": "CreateRouteTable",
                "State": "active"
            }
        ],
        "Tags": [
            {
                "Key": "Name",
                "Value": "datacenter-pub-rt"
            }
        ],
        "VpcId": "vpc-020c460f013c6d7a2",
        "OwnerId": "495779504297"
    }
}
```

**Public Route Table ID:** `rtb-00410046bcf546ea9`

#### Add Default Route to IGW

```bash
aws ec2 create-route \
  --route-table-id rtb-00410046bcf546ea9 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-03c237657533926b7 \
  --region us-east-1
```

**Output:**

```json
{
    "Return": true
}
```

#### Associate Route Table with Public Subnet

```bash
aws ec2 associate-route-table \
  --route-table-id rtb-00410046bcf546ea9 \
  --subnet-id subnet-0a8ea30f1103e6408 \
  --region us-east-1
```

**Output:**

```json
{
    "AssociationId": "rtbassoc-062ac268df09288cb",
    "AssociationState": {
        "State": "associated"
    }
}
```

***

### Phase 4: Security Group for NAT Instance

#### Create Security Group

```bash
aws ec2 create-security-group \
  --group-name datacenter-nat-sg \
  --description "Security group for NAT instance" \
  --vpc-id vpc-020c460f013c6d7a2 \
  --region us-east-1 \
  --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=datacenter-nat-sg}]'
```

**Output:**

```json
{
    "GroupId": "sg-04d07cfadbb21781e",
    "Tags": [
        {
            "Key": "Name",
            "Value": "datacenter-nat-sg"
        }
    ],
    "SecurityGroupArn": "arn:aws:ec2:us-east-1:495779504297:security-group/sg-04d07cfadbb21781e"
}
```

**Security Group ID:** `sg-04d07cfadbb21781e`

#### Add Inbound Rules

```bash
# Allow ALL traffic from the private subnet
aws ec2 authorize-security-group-ingress \
  --group-id sg-04d07cfadbb21781e \
  --protocol -1 \
  --cidr 10.1.1.0/24 \
  --region us-east-1
```

**Output:**

```json
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0aae0ca496017bccb",
            "GroupId": "sg-04d07cfadbb21781e",
            "GroupOwnerId": "495779504297",
            "IsEgress": false,
            "IpProtocol": "-1",
            "FromPort": -1,
            "ToPort": -1,
            "CidrIpv4": "10.1.1.0/24"
        }
    ]
}
```

```bash
# Allow HTTPS from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-04d07cfadbb21781e \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0 \
  --region us-east-1
```

**Output:**

```json
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-04a5c13b6a2d0f91b",
            "GroupId": "sg-04d07cfadbb21781e",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 443,
            "ToPort": 443,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
```

```bash
# Allow HTTP from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-04d07cfadbb21781e \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0 \
  --region us-east-1
```

**Output:**

```json
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-098ddf021394544bf",
            "GroupId": "sg-04d07cfadbb21781e",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
```

***

### Phase 5: Launch the NAT Instance

#### Get Latest Amazon Linux 2023 AMI

```bash
aws ec2 describe-images \
  --owners amazon \
  --filters \
    "Name=name,Values=al2023-ami-*-x86_64" \
    "Name=state,Values=available" \
  --query "sort_by(Images, &CreationDate)[-1].ImageId" \
  --output text \
  --region us-east-1
```

**Output:**

```
ami-0fc6cf99992956a4a
```

#### Create User Data Script

This script runs on first boot and configures the instance as a NAT device:

```bash
cat <<'EOF' > nat-userdata.sh
#!/bin/bash
# Enable IP forwarding immediately
echo 1 > /proc/sys/net/ipv4/ip_forward

# Make IP forwarding persistent across reboots
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# Install iptables (not included by default on Amazon Linux 2023)
dnf install -y iptables-services

# Enable and start iptables service
systemctl enable iptables
systemctl start iptables

# Detect the primary network interface
INTERFACE=$(ip route | grep default | awk '{print $5}')

# Configure NAT masquerading — translates private IPs to the instance's public IP
iptables -t nat -A POSTROUTING -o $INTERFACE -j MASQUERADE

# Allow all forwarded traffic
iptables -F FORWARD
iptables -A FORWARD -j ACCEPT

# Save rules so they persist across reboots
service iptables save
EOF
```

#### Launch the NAT Instance

> ⚠️ The `--source-dest-check` flag is **not** supported in `run-instances`. It must be disabled separately using `modify-instance-attribute` after launch.

```bash
aws ec2 run-instances \
  --image-id ami-0fc6cf99992956a4a \
  --instance-type t2.micro \
  --subnet-id subnet-0a8ea30f1103e6408 \
  --security-group-ids sg-04d07cfadbb21781e \
  --associate-public-ip-address \
  --user-data file://nat-userdata.sh \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=datacenter-nat-instance}]' \
  --region us-east-1
```

**Output (truncated):**

```json
{
    "ReservationId": "r-08ceefd48c3a6ece5",
    "OwnerId": "495779504297",
    "Instances": [
        {
            "InstanceId": "i-02ab9f50e649e277e",
            "ImageId": "ami-0fc6cf99992956a4a",
            "InstanceType": "t2.micro",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "PrivateIpAddress": "10.1.2.103",
            "SubnetId": "subnet-0a8ea30f1103e6408",
            "VpcId": "vpc-020c460f013c6d7a2",
            "SourceDestCheck": true,
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "datacenter-nat-instance"
                }
            ],
            "Placement": {
                "AvailabilityZone": "us-east-1a"
            }
        }
    ]
}
```

**NAT Instance ID:** `i-02ab9f50e649e277e`

#### Wait for Instance to Be Running

```bash
aws ec2 wait instance-running \
  --instance-ids i-02ab9f50e649e277e \
  --region us-east-1
```

_(No output — command returns when instance is in `running` state)_

#### Disable Source/Destination Check

This is **critical** for NAT to work. By default, EC2 drops packets not destined for its own IP. Disabling this check allows the instance to forward traffic on behalf of others.

```bash
aws ec2 modify-instance-attribute \
  --instance-id i-02ab9f50e649e277e \
  --no-source-dest-check \
  --region us-east-1
```

_(No output — success is silent)_

***

### Phase 6: Update Private Subnet Route Table

Add a default route in the **private** subnet's route table pointing all internet-bound traffic (`0.0.0.0/0`) to the NAT instance.

```bash
aws ec2 create-route \
  --route-table-id rtb-0e9bf061b3aa4ca15 \
  --destination-cidr-block 0.0.0.0/0 \
  --instance-id i-02ab9f50e649e277e \
  --region us-east-1
```

**Output:**

```json
{
    "Return": true
}
```

***

### Phase 7: Verify the Setup

Wait approximately 60 seconds for the cron job on `datacenter-priv-ec2` to run, then check the S3 bucket:

```bash
aws s3 ls s3://datacenter-nat-17811 --region us-east-1
```

**Output:**

```
2026-03-23 07:21:09         21 datacenter-test.txt
```

✅ **Lab Complete!** The file `datacenter-test.txt` is present in the S3 bucket, confirming that the private EC2 instance successfully reached the internet through the NAT instance.

***

### Key Concepts

#### Why NAT Instance Instead of NAT Gateway?

| Feature      | NAT Instance               | NAT Gateway                 |
| ------------ | -------------------------- | --------------------------- |
| Cost         | Pay only for EC2 (cheaper) | Hourly + data transfer fees |
| Management   | Manual (you manage the OS) | Fully managed by AWS        |
| Availability | Single point of failure    | Highly available            |
| Bandwidth    | Limited by instance type   | Scales automatically        |
| Use case     | Dev/test, cost-sensitive   | Production workloads        |

#### Why Disable Source/Dest Check?

By default, every EC2 instance checks that packets it receives are addressed to **its own IP**. A NAT instance must forward packets on behalf of private instances — those packets have source IPs belonging to other instances. Disabling this check tells the instance to process all packets regardless of destination.

#### How iptables MASQUERADE Works

```
Private EC2 (10.1.1.x) → sends packet to S3
   ↓
NAT Instance receives packet with source=10.1.1.x
   ↓
iptables MASQUERADE rewrites source IP to NAT's public IP
   ↓
Packet goes out to internet with NAT's IP as source
   ↓
Response comes back to NAT's public IP
   ↓
iptables rewrites destination back to 10.1.1.x
   ↓
Private EC2 receives the response
```

#### Why iptables Must Be Installed on AL2023

Amazon Linux 2023 uses `nftables` as its default firewall framework and does **not** include `iptables` out of the box. You must explicitly install `iptables-services` via `dnf` before you can configure NAT masquerading rules.

***

### Resource Summary

| Resource            | Name                      | ID                         |
| ------------------- | ------------------------- | -------------------------- |
| VPC                 | `datacenter-priv-vpc`     | `vpc-020c460f013c6d7a2`    |
| Private Subnet      | `datacenter-priv-subnet`  | `subnet-0010f478711a26440` |
| Public Subnet       | `datacenter-pub-subnet`   | `subnet-0a8ea30f1103e6408` |
| Internet Gateway    | `datacenter-igw`          | `igw-03c237657533926b7`    |
| Public Route Table  | `datacenter-pub-rt`       | `rtb-00410046bcf546ea9`    |
| Private Route Table | _(existing)_              | `rtb-0e9bf061b3aa4ca15`    |
| Security Group      | `datacenter-nat-sg`       | `sg-04d07cfadbb21781e`     |
| NAT Instance        | `datacenter-nat-instance` | `i-02ab9f50e649e277e`      |
| AMI                 | Amazon Linux 2023         | `ami-0fc6cf99992956a4a`    |
| S3 Bucket           | `datacenter-nat-17811`    | —                          |

***

<figure><img src=".gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/Screenshot 2026-03-23 125419.png" alt=""><figcaption></figcaption></figure>


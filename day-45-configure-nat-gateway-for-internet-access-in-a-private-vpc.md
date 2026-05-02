# Day 45: Configure NAT Gateway for Internet Access in a Private VPC

The Nautilus DevOps team is tasked with enabling internet access for an EC2 instance running in a private subnet. This instance should be able to upload a test file to a public S3 bucket once it can access the internet. To achieve this, the team must set up a NAT Gateway in a public subnet within the same VPC.

1\) A VPC named `xfusion-priv-vpc` and a private subnet `xfusion-priv-subnet` have already been created.
2\) An EC2 instance named `xfusion-priv-ec2` is already running in the private subnet.
3\) The EC2 instance is configured with a cron job that uploads a test file to a bucket `xfusion-nat-484948118` once internet is accessible.

Your task is to:

* Create a public subnet named `xfusion-pub-subnet` in the same VPC.
* Create an Internet Gateway and attach it to the VPC.
* Create a route table `xfusion-pub-rt` and associate it with the public subnet.
* Allocate an Elastic IP and create a NAT Gateway named `xfusion-natgw`.
* Update the private route table to route 0.0.0.0/0 traffic via the NAT Gateway.

Once complete, verify that the EC2 instance can reach the internet by confirming the presence of the test file in the S3 bucket `xfusion-nat-484948118`. After completing all the configuration, please wait a few minutes for the test file to appear in the bucket, as it may take `2–3 minutes`.

**Notes:**

* Use region `us-east-1`
* To show/hide terminal: use the panel toggle button.

## AWS NAT Gateway — Enable Internet Access for Private EC2 Instance

### Overview

This lab documents how to enable internet access for an EC2 instance running in a private subnet by setting up a NAT Gateway. Once internet access is established, the EC2 instance uploads a test file to a public S3 bucket via a pre-configured cron job.

***

### Lab Scenario

**Given:**

* A VPC named `nautilus-priv-vpc` and a private subnet `nautilus-priv-subnet` already exist.
* An EC2 instance named `nautilus-priv-ec2` is already running in the private subnet.
* The EC2 instance has a cron job that uploads a test file to S3 bucket `nautilus-nat-528694802` once internet is accessible.

**Tasks:**

1. Create a public subnet named `nautilus-pub-subnet` in the same VPC.
2. Create an Internet Gateway and attach it to the VPC.
3. Create a route table `nautilus-pub-rt` and associate it with the public subnet.
4. Allocate an Elastic IP and create a NAT Gateway named `nautilus-natgw`.
5. Create a **dedicated private route table**, explicitly associate the private subnet, and route `0.0.0.0/0` traffic via the NAT Gateway.
6. Verify by confirming the test file appears in the S3 bucket.

> **Region:** `us-east-1`

***

### Architecture

```bash
Internet
    │
    ▼
Internet Gateway (nautilus-igw)
    │
    ▼
Public Subnet (nautilus-pub-subnet 10.1.2.0/24)
    │   nautilus-pub-rt → 0.0.0.0/0 → IGW
    │
    ▼
NAT Gateway (nautilus-natgw) ← Elastic IP
    │
    ▼
Private Subnet (nautilus-priv-subnet 10.1.1.0/24)
    │   nautilus-priv-rt → 0.0.0.0/0 → NAT GW
    │
    ▼
EC2 Instance (nautilus-priv-ec2)
    │
    ▼
S3 Bucket (nautilus-nat-528694802)
```

***

### Solution

#### Step 1 — Set Region and Gather Existing Resource Info

```bash
export AWS_DEFAULT_REGION=us-east-1

# Get VPC ID
VPC_ID=$(aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=nautilus-priv-vpc" \
  --query "Vpcs[0].VpcId" --output text)
echo "VPC_ID: $VPC_ID"

# Get private subnet ID and AZ
PRIV_SUBNET_ID=$(aws ec2 describe-subnets \
  --filters "Name=tag:Name,Values=nautilus-priv-subnet" \
  --query "Subnets[0].SubnetId" --output text)
echo "PRIV_SUBNET_ID: $PRIV_SUBNET_ID"

PRIV_AZ=$(aws ec2 describe-subnets \
  --subnet-ids $PRIV_SUBNET_ID \
  --query "Subnets[0].AvailabilityZone" --output text)
echo "PRIV_AZ: $PRIV_AZ"

# Get VPC CIDR to derive public subnet CIDR
VPC_CIDR=$(aws ec2 describe-vpcs --vpc-ids $VPC_ID \
  --query "Vpcs[0].CidrBlock" --output text)
echo "VPC_CIDR: $VPC_CIDR"
```

**Output:**

```
VPC_ID: vpc-0a8ffd6372400d0ab
PRIV_SUBNET_ID: subnet-02b048ee2a61fc4aa
PRIV_AZ: us-east-1a
VPC_CIDR: 10.1.0.0/16
```

***

#### Step 2 — Create the Public Subnet

The VPC CIDR is `10.1.0.0/16` and the private subnet uses `10.1.1.0/24`, so the public subnet gets `10.1.2.0/24`.

```bash
# Derive public CIDR from VPC base
BASE=$(echo $VPC_CIDR | cut -d'.' -f1-2)
PUB_CIDR="${BASE}.2.0/24"
echo "Public CIDR will be: $PUB_CIDR"

# Create public subnet
PUB_SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block $PUB_CIDR \
  --availability-zone $PRIV_AZ \
  --query "Subnet.SubnetId" --output text)

# Tag it
aws ec2 create-tags --resources $PUB_SUBNET_ID \
  --tags Key=Name,Value=nautilus-pub-subnet

# Enable auto-assign public IP
aws ec2 modify-subnet-attribute \
  --subnet-id $PUB_SUBNET_ID \
  --map-public-ip-on-launch

echo "Public Subnet ID: $PUB_SUBNET_ID"
```

**Output:**

```
Public CIDR will be: 10.1.2.0/24
Public Subnet ID: subnet-0e033014380582eae
```

***

#### Step 3 — Create and Attach an Internet Gateway

```bash
IGW_ID=$(aws ec2 create-internet-gateway \
  --query "InternetGateway.InternetGatewayId" --output text)

aws ec2 create-tags --resources $IGW_ID \
  --tags Key=Name,Value=nautilus-igw

aws ec2 attach-internet-gateway \
  --internet-gateway-id $IGW_ID \
  --vpc-id $VPC_ID

echo "IGW_ID: $IGW_ID"
```

**Output:**

```
IGW_ID: igw-026c4b2002d8a4b45
```

***

#### Step 4 — Create Public Route Table and Associate with Public Subnet

```bash
PUB_RT_ID=$(aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --query "RouteTable.RouteTableId" --output text)

aws ec2 create-tags --resources $PUB_RT_ID \
  --tags Key=Name,Value=nautilus-pub-rt

# Route all internet traffic to the IGW
aws ec2 create-route \
  --route-table-id $PUB_RT_ID \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id $IGW_ID

# Associate with public subnet
aws ec2 associate-route-table \
  --route-table-id $PUB_RT_ID \
  --subnet-id $PUB_SUBNET_ID

echo "Public RT ID: $PUB_RT_ID"
```

**Output:**

```
{
    "Return": true
}
{
    "AssociationId": "rtbassoc-073022f64f7fc5a41",
    "AssociationState": {
        "State": "associated"
    }
}
Public RT ID: rtb-0ee7ecafe893dbc64
```

***

#### Step 5 — Allocate Elastic IP and Create NAT Gateway

```bash
EIP_ALLOC=$(aws ec2 allocate-address \
  --domain vpc --query "AllocationId" --output text)
echo "EIP_ALLOC: $EIP_ALLOC"

NAT_GW_ID=$(aws ec2 create-nat-gateway \
  --subnet-id $PUB_SUBNET_ID \
  --allocation-id $EIP_ALLOC \
  --query "NatGateway.NatGatewayId" --output text)

aws ec2 create-tags --resources $NAT_GW_ID \
  --tags Key=Name,Value=nautilus-natgw

echo "NAT_GW_ID: $NAT_GW_ID"

# Wait until NAT Gateway is fully available before proceeding
echo "Waiting for NAT Gateway to become available..."
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GW_ID
echo "NAT Gateway is ready!"
```

**Output:**

```
EIP_ALLOC: eipalloc-0ee68ba55d33534bf
NAT_GW_ID: nat-076bd018f26356a0a
Waiting for NAT Gateway to become available...
NAT Gateway is ready!
```

***

#### Step 6 — Create Dedicated Private Route Table and Associate with Private Subnet

> **Critical:** Do NOT use the main VPC route table. The lab validator requires an **explicit association** between the private subnet and a dedicated private route table. Using the main/default route table will cause the lab check to fail even if routing works correctly.

```bash
# Create a new dedicated private route table
PRIV_RT_ID=$(aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --query "RouteTable.RouteTableId" --output text)

aws ec2 create-tags --resources $PRIV_RT_ID \
  --tags Key=Name,Value=nautilus-priv-rt

echo "Private RT ID: $PRIV_RT_ID"

# Explicitly associate the private subnet with this route table
aws ec2 associate-route-table \
  --route-table-id $PRIV_RT_ID \
  --subnet-id $PRIV_SUBNET_ID

echo "Private subnet explicitly associated!"

# Add 0.0.0.0/0 route via NAT Gateway
aws ec2 create-route \
  --route-table-id $PRIV_RT_ID \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id $NAT_GW_ID

echo "NAT route added to private route table!"
```

**Output:**

```
Private RT ID: rtb-079651d4b3265d552
{
    "AssociationId": "rtbassoc-01f937529f7a5a3bc",
    "AssociationState": {
        "State": "associated"
    }
}
Private subnet explicitly associated!
{
    "Return": true
}
NAT route added to private route table!
```

***

#### Step 7 — Verify Routes and Associations

```bash
echo "=== Private Route Table Routes ==="
aws ec2 describe-route-tables --route-table-ids $PRIV_RT_ID \
  --query "RouteTables[0].Routes" --output table

echo "=== Private Subnet Association ==="
aws ec2 describe-route-tables --route-table-ids $PRIV_RT_ID \
  --query "RouteTables[0].Associations" --output table
```

**Output:**

```bash
=== Private Route Table Routes ===
----------------------------------------------------------------------------------------------
|                                     DescribeRouteTables                                    |
+-----------------------+------------+-------------------------+-------------------+---------+
| DestinationCidrBlock  | GatewayId  |      NatGatewayId       |      Origin       |  State  |
+-----------------------+------------+-------------------------+-------------------+---------+
|  10.1.0.0/16          |  local     |                         |  CreateRouteTable |  active |
|  0.0.0.0/0            |            |  nat-076bd018f26356a0a  |  CreateRoute      |  active |
+-----------------------+------------+-------------------------+-------------------+---------+

=== Private Subnet Association ===
----------------------------------------------------------------------------------------------
|                                     DescribeRouteTables                                    |
+------+------------------------------+-------------------------+----------------------------+
| Main |   RouteTableAssociationId    |      RouteTableId       |         SubnetId           |
+------+------------------------------+-------------------------+----------------------------+
|False |  rtbassoc-01f937529f7a5a3bc  |  rtb-079651d4b3265d552  |  subnet-02b048ee2a61fc4aa  |
+------+------------------------------+-------------------------+----------------------------+
```

***

#### Step 8 — Verify S3 Upload (Wait 2–3 minutes)

```bash
echo "Waiting 3 minutes for cron job to upload test file..."
sleep 180
echo "=== S3 Bucket Contents ==="
aws s3 ls s3://nautilus-nat-528694802/
```

**Output:**

```
=== S3 Bucket Contents ===
2026-04-26 15:25:03          0 nautilus-test.txt
```

* The test file `nautilus-test.txt` is present in the bucket, confirming the EC2 instance in the private subnet can reach the internet via the NAT Gateway.

***

### Resources Created

| Resource            | Name                  | ID                           |
| ------------------- | --------------------- | ---------------------------- |
| Public Subnet       | `nautilus-pub-subnet` | `subnet-0e033014380582eae`   |
| Internet Gateway    | `nautilus-igw`        | `igw-026c4b2002d8a4b45`      |
| Public Route Table  | `nautilus-pub-rt`     | `rtb-0ee7ecafe893dbc64`      |
| Elastic IP          | —                     | `eipalloc-0ee68ba55d33534bf` |
| NAT Gateway         | `nautilus-natgw`      | `nat-076bd018f26356a0a`      |
| Private Route Table | `nautilus-priv-rt`    | `rtb-079651d4b3265d552`      |

***

### Key Lessons

#### What Fails the Lab Checker

Using the **main VPC route table** for the private subnet — even though traffic routes correctly, the lab validator checks for an explicit subnet-to-route-table association.

```bash
# This returns "None" if the private subnet has no explicit association
PRIV_RT_ID=$(aws ec2 describe-route-tables \
  --filters "Name=association.subnet-id,Values=$PRIV_SUBNET_ID" \
  --query "RouteTables[0].RouteTableId" --output text)
# Result: None  ← lab will fail
```

#### What Passes the Lab Checker

Create a **new dedicated route table**, **explicitly associate** the private subnet to it, then add the NAT Gateway route.

```bash
# Main = False confirms it's an explicit (non-default) association
| Main |   RouteTableAssociationId    |  SubnetId                  |
|False |  rtbassoc-01f937529f7a5a3bc  |  subnet-02b048ee2a61fc4aa  |
```

#### CIDR Selection

Always check the VPC CIDR before creating subnets. If the VPC is `10.1.0.0/16` and the private subnet is `10.1.1.0/24`, use `10.1.2.0/24` for the public subnet — not `10.0.2.0/24`.

```bash
# Safe way to derive the public subnet CIDR automatically
BASE=$(echo $VPC_CIDR | cut -d'.' -f1-2)
PUB_CIDR="${BASE}.2.0/24"
```

#### NAT Gateway Must Be in the Public Subnet

The NAT Gateway must be placed in the **public subnet** (which has an IGW route), not the private subnet. This is what allows it to forward traffic to the internet on behalf of private instances.

<figure><img src=".gitbook/assets/image (108).png" alt=""><figcaption></figcaption></figure>

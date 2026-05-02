# Day 27: Configuring a Public VPC with an EC2 Instance for Internet Access

## **Creating a Public VPC and EC2 Instance on AWS**

***

### **Question / Task**

> The Networking Team requested a public VPC to host public-facing services.
> **Requirements:**
>
> 1. Create a VPC named `xfusion-pub-vpc`.
> 2. Create a public subnet `xfusion-pub-subnet` with automatic public IP assignment.
> 3. Attach an Internet Gateway.
> 4. Create a route table with a default route to the IGW and associate it with the subnet.
> 5. Launch a **t2.micro** EC2 instance `xfusion-pub-ec2` in this subnet with SSH port 22 open.
> 6. Verify internet connectivity.

***

### **Step 1: Create the VPC**

**Command:**

```bash
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=xfusion-pub-vpc}]'
```

**Output:**

```json
{
    "Vpc": {
        "VpcId": "vpc-08814bd2a2571d089",
        "State": "pending",
        "CidrBlock": "10.0.0.0/16",
        "Tags": [{"Key": "Name","Value": "xfusion-pub-vpc"}]
    }
}
```

**Explanation:**
This creates a new VPC with CIDR `10.0.0.0/16` and tags it `xfusion-pub-vpc`.

***

### **Step 2: Create a Public Subnet**

**Command:**

```bash
aws ec2 create-subnet \
  --vpc-id vpc-08814bd2a2571d089 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=xfusion-pub-subnet}]'
```

**Output:**

```json
{
    "Subnet": {
        "SubnetId": "subnet-05125bf320a7e1d33",
        "State": "available",
        "VpcId": "vpc-08814bd2a2571d089",
        "CidrBlock": "10.0.1.0/24",
        "MapPublicIpOnLaunch": false
    }
}
```

**Explanation:**
A subnet is created. By default, AWS does **not** assign public IPs. We fix that in the next step.

***

### **Step 3: Enable Auto-Assign Public IP**

**Command:**

```bash
aws ec2 modify-subnet-attribute \
  --subnet-id subnet-05125bf320a7e1d33 \
  --map-public-ip-on-launch
```

**Explanation:**
All instances launched in this subnet will automatically receive a public IP.

***

### **Step 4: Create an Internet Gateway**

**Command:**

```bash
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=xfusion-igw}]'
```

**Output:**

```json
{
    "InternetGateway": {
        "InternetGatewayId": "igw-04e0a390f6d9fbffb",
        "Tags": [{"Key":"Name","Value":"xfusion-igw"}]
    }
}
```

**Explanation:**
This creates an IGW to enable internet access.

***

### **Step 5: Attach IGW to the VPC**

**Command:**

```bash
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-04e0a390f6d9fbffb \
  --vpc-id vpc-08814bd2a2571d089
```

**Explanation:**
Now the VPC can route traffic to the internet.

***

### **Step 6: Create a Route Table and Route**

**Create Route Table:**

```bash
aws ec2 create-route-table \
  --vpc-id vpc-08814bd2a2571d089
```

**Output:**

```json
{
    "RouteTable": {
        "RouteTableId": "rtb-0173021281c1b4196",
        "VpcId": "vpc-08814bd2a2571d089"
    }
}
```

**Add Default Route to IGW:**

```bash
aws ec2 create-route \
  --route-table-id rtb-0173021281c1b4196 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-04e0a390f6d9fbffb
```

**Associate Route Table with Subnet:**

```bash
aws ec2 associate-route-table \
  --subnet-id subnet-05125bf320a7e1d33 \
  --route-table-id rtb-0173021281c1b4196
```

**Output:**

```json
{
    "AssociationId": "rtbassoc-09383d82d6f06f02d",
    "AssociationState": {"State": "associated"}
}
```

**Explanation:**
This makes the subnet a **public subnet**, allowing instances to access the internet.

***

### **Step 7: Create Security Group and Open SSH**

**Create SG:**

```bash
aws ec2 create-security-group \
  --group-name xfusion-pub-sg \
  --description "Allow SSH" \
  --vpc-id vpc-08814bd2a2571d089
```

**Output:**

```json
{
    "GroupId": "sg-05596d0831e3c2a9f"
}
```

**Allow SSH (Port 22):**

```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-05596d0831e3c2a9f \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

**Explanation:**
This allows SSH from anywhere for management.

***

### **Step 8: Create a Key Pair**

```bash
aws ec2 create-key-pair \
  --key-name xfusion-key \
  --query 'KeyMaterial' \
  --output text > xfusion-key.pem
chmod 400 xfusion-key.pem
```

**Explanation:**
This creates an SSH key to access the EC2 instance.

***

### **Step 9: Launch EC2 Instance**

```bash
aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \
  --instance-type t2.micro \
  --key-name xfusion-key \
  --subnet-id subnet-05125bf320a7e1d33 \
  --security-group-ids sg-05596d0831e3c2a9f \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-pub-ec2}]'
```

**Output:**

```json
{
    "Instances": [
        {
            "InstanceId": "i-04816ecabdd5b212a",
            "State": {"Name": "pending"},
            "SubnetId": "subnet-05125bf320a7e1d33",
            "SecurityGroups": [{"GroupId": "sg-05596d0831e3c2a9f"}]
        }
    ]
}
```

**Explanation:**
The EC2 instance is launched in the public subnet with SSH access enabled.

***

### **Step 10: Verify Public IP**

```bash
aws ec2 describe-instances \
  --instance-ids i-04816ecabdd5b212a \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text
```

**Output:**

```
3.239.104.156
```

***

### **Step 11: Connect via SSH and Test Internet**

```bash
ssh -i xfusion-key.pem ec2-user@3.239.104.156
ping -c 4 8.8.8.8
```

**Output:**

```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=2.06 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=1.50 ms
...
```

**Explanation:**
SSH works and instance can reach the internet.

***

### **Step 12: Summary**

| Resource         | ID / Name             | Status             |
| ---------------- | --------------------- | ------------------ |
| VPC              | `xfusion-pub-vpc`     | Available          |
| Subnet           | `xfusion-pub-subnet`  | Public, Auto-IP    |
| Internet Gateway | `xfusion-igw`         | Attached           |
| Route Table      | rtb-0173021281c1b4196 | 0.0.0.0/0 → `IGW`  |
| Security Group   | `xfusion-pub-sg`      | SSH open           |
| EC2 Instance     | xfusion-pub-ec2       | Running, Public IP |
| Key Pair         | `xfusion-key`         | Ready for SSH      |

***

 **The public-facing VPC environment is ready for applications.**

***

<figure><img src=".gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

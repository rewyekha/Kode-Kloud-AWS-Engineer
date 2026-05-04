# Day 31: Configuring a Private RDS Instance for Application Development

## AWS RDS: Provisioning a Private MySQL Instance for Application Development

> **Platform:** KodeKloud | **Cloud:** AWS | **Region:** us-east-1 **Difficulty:** Intermediate | **Topic:** AWS RDS, MySQL, VPC, Storage Autoscaling, CLI

***

### Table of Contents

1. [Lab Overview](#lab-overview)
2. [Lab Objectives](#lab-objectives)
3. [Prerequisites](#prerequisites)
4. [Architecture](#architecture)
5. [Phase 1: Set Environment Variables](#phase-1-set-environment-variables)
6. [Phase 2: Gather VPC and Subnet Information](#phase-2-gather-vpc-and-subnet-information)
7. [Phase 3: Create DB Subnet Group](#phase-3-create-db-subnet-group)
8. [Phase 4: Create Security Group for RDS](#phase-4-create-security-group-for-rds)
9. [Phase 5: Create the RDS Instance](#phase-5-create-the-rds-instance)
10. [Phase 6: Wait for Instance to Become Available](#phase-6-wait-for-instance-to-become-available)
11. [Phase 7: Validate the RDS Instance](#phase-7-validate-the-rds-instance)
12. [Summary](#summary)
13. [Key Concepts](#key-concepts)
14. [Resource Reference](#resource-reference)

***

### Lab Overview

The Nautilus Development Team is working on a new application feature that requires a reliable and scalable database solution. To facilitate development and testing, they need a new private RDS instance. This instance will be used to store critical application data and must be provisioned using the AWS free tier to minimize costs during the initial development phase. The team has chosen MySQL as the database engine due to its compatibility with their existing systems.

The DevOps team has been tasked with setting up this RDS instance, ensuring that it is correctly configured and available for use by the development team.

***

### Lab Objectives

1. **Provision a Private RDS Instance:** Create a new private RDS instance named `devops-rds` using a `sandbox` template with a `db.t3.micro` instance type.
2. **Engine Configuration:** Use the `MySQL` engine with version `8.4.x`.
3. **Enable Storage Autoscaling:** Enable storage autoscaling and set the threshold value to `50GB`. Keep the rest of the configurations as default.
4. **Instance Availability:** Ensure the instance is in the `available` state before submitting the task.

***

### Prerequisites

| Field         | Value                                                                 |
| ------------- | --------------------------------------------------------------------- |
> **Note:** Run `showcreds` on the `aws-client` host to retrieve temporary AWS credentials, then configure the AWS CLI using `aws configure`.
| Access Method | AWS CLI on `aws-client` host                                          |

***

### Architecture

```sh
┌─────────────────────────────────────────────────────────────────────┐
│                        AWS us-east-1                                │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │               Default VPC (vpc-0b73f32d24d9f9625)           │   │
│   │                                                             │   │
│   │   ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐     │   │
│   │   │us-east-1a│ │us-east-1b│ │us-east-1c│ │us-east-1d│ ... │   │
│   │   │  subnet  │ │  subnet  │ │  subnet  │ │  subnet  │     │   │
│   │   └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘     │   │
│   │        └────────────┴────────────┴─────────────┘           │   │
│   │                          |                                  │   │
│   │               db-subnet-group                               │   │
│   │                          |                                  │   │
│   │              ┌───────────────────────┐                      │   │
│   │              │      devops-rds        │                      │   │
│   │              │   MySQL 8.4.3          │                      │   │
│   │              │   db.t3.micro          │                      │   │
│   │              │   Storage: 20GB        │                      │   │
│   │              │   Max Storage: 50GB    │                      │   │
│   │              │   Public: false        │                      │   │
│   │              │   Status: available    │                      │   │
│   │              └───────────────────────┘                      │   │
│   │                          |                                  │   │
│   │              ┌───────────────────────┐                      │   │
│   │              │  Security Group        │                      │   │
│   │              │  rds-sg                │                      │   │
│   │              │  sg-0c472975a9422f2ea  │                      │   │
│   │              └───────────────────────┘                      │   │
│   └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

***

### Phase 1: Set Environment Variables

Define all configuration values as shell variables upfront to keep subsequent commands clean and reusable.

```bash
RDS_NAME="devops-rds"
INSTANCE_TYPE="db.t3.micro"
ENGINE="mysql"
ENGINE_VERSION="8.4.3"
MAX_STORAGE=50
SUBNET_GROUP="db-subnet-group"
SECURITY_GROUP="rds-sg"
```

**Terminal Output:**

```bash
~ on ☁️  (us-east-1) ➜  RDS_NAME="devops-rds"
INSTANCE_TYPE="db.t3.micro"
ENGINE="mysql"
ENGINE_VERSION="8.4.3"
MAX_STORAGE=50
SUBNET_GROUP="db-subnet-group"
SECURITY_GROUP="rds-sg"

~ on ☁️  (us-east-1) ➜
```

***

### Phase 2: Gather VPC and Subnet Information

#### Step 1: Get the Default VPC ID

RDS requires a VPC to be deployed into. Retrieve the default VPC ID and store it in a variable:

```bash
VPC_ID=$(aws ec2 describe-vpcs \
  --filters Name=isDefault,Values=true \
  --query "Vpcs[0].VpcId" \
  --output text \
  --region us-east-1)

echo $VPC_ID
```

**Terminal Output:**

```bash
~ on ☁️  (us-east-1) ➜  VPC_ID=$(aws ec2 describe-vpcs \
  --filters Name=isDefault,Values=true \
  --query "Vpcs[0].VpcId" \
  --output text \
  --region us-east-1)

echo $VPC_ID
vpc-0b73f32d24d9f9625

~ on ☁️  (us-east-1) ➜
```

#### Step 2: Get All Subnets in the Default VPC

RDS DB Subnet Groups require subnets from at least 2 different Availability Zones. Collect all subnet IDs from the default VPC:

```bash
SUBNET_IDS=$(aws ec2 describe-subnets \
  --filters Name=vpc-id,Values="$VPC_ID" \
  --query "Subnets[*].SubnetId" \
  --output text \
  --region us-east-1)

echo $SUBNET_IDS
```

**Terminal Output:**

```bash
~ on ☁️  (us-east-1) ➜  SUBNET_IDS=$(aws ec2 describe-subnets \
  --filters Name=vpc-id,Values="$VPC_ID" \
  --query "Subnets[*].SubnetId" \
  --output text \
  --region us-east-1)

echo $SUBNET_IDS
subnet-0968e1f342f7d5e00 subnet-0f1f9b4afc216545c subnet-0480cba21faebe63f subnet-0f42654d7f527bbc2 subnet-0637f846e34e2218e subnet-08f5b39e48ba03c99

~ on ☁️  (us-east-1) ➜
```

Six subnets were found across six Availability Zones: `us-east-1a` through `us-east-1f`.

***

### Phase 3: Create DB Subnet Group

A DB Subnet Group tells RDS which subnets it is allowed to use when placing the database instance. This is a required component before an RDS instance can be created inside a VPC.

```bash
aws rds create-db-subnet-group \
  --db-subnet-group-name $SUBNET_GROUP \
  --db-subnet-group-description "Subnet group for RDS" \
  --subnet-ids $SUBNET_IDS \
  --region us-east-1
```

**Terminal Output:**

```bash
~ on ☁️  (us-east-1) ➜  aws rds create-db-subnet-group \
  --db-subnet-group-name $SUBNET_GROUP \
  --db-subnet-group-description "Subnet group for RDS" \
  --subnet-ids $SUBNET_IDS \
  --region us-east-1
{
    "DBSubnetGroup": {
        "DBSubnetGroupName": "db-subnet-group",
        "DBSubnetGroupDescription": "Subnet group for RDS",
        "VpcId": "vpc-0b73f32d24d9f9625",
        "SubnetGroupStatus": "Complete",
        "Subnets": [
            {
                "SubnetIdentifier": "subnet-0968e1f342f7d5e00",
                "SubnetAvailabilityZone": {
                    "Name": "us-east-1e"
                },
                "SubnetOutpost": {},
                "SubnetStatus": "Active"
            },
            {
                "SubnetIdentifier": "subnet-0f1f9b4afc216545c",
                "SubnetAvailabilityZone": {
                    "Name": "us-east-1f"
                },
                "SubnetOutpost": {},
                "SubnetStatus": "Active"
            },
            {
                "SubnetIdentifier": "subnet-0480cba21faebe63f",
                "SubnetAvailabilityZone": {
                    "Name": "us-east-1a"
                },
                "SubnetOutpost": {},
                "SubnetStatus": "Active"
            },
            {
                "SubnetIdentifier": "subnet-0f42654d7f527bbc2",
                "SubnetAvailabilityZone": {
                    "Name": "us-east-1c"
                },
                "SubnetOutpost": {},
                "SubnetStatus": "Active"
            },
            {
                "SubnetIdentifier": "subnet-0637f846e34e2218e",
                "SubnetAvailabilityZone": {
                    "Name": "us-east-1b"
                },
                "SubnetOutpost": {},
                "SubnetStatus": "Active"
            },
            {
                "SubnetIdentifier": "subnet-08f5b39e48ba03c99",
                "SubnetAvailabilityZone": {
                    "Name": "us-east-1d"
                },
                "SubnetOutpost": {},
                "SubnetStatus": "Active"
            }
        ],
        "DBSubnetGroupArn": "arn:aws:rds:us-east-1:467239208986:subgrp:db-subnet-group",
        "SupportedNetworkTypes": [
            "IPV4"
        ]
    }
}

~ on ☁️  (us-east-1) ➜
```

The subnet group `db-subnet-group` was created with `SubnetGroupStatus: Complete` and all 6 subnets active across 6 Availability Zones.

***

### Phase 4: Create Security Group for RDS

A dedicated security group is created for the RDS instance. This allows fine-grained control over which resources are permitted to connect to the database.

```bash
RDS_SG_ID=$(aws ec2 create-security-group \
  --group-name $SECURITY_GROUP \
  --description "RDS SG" \
  --vpc-id "$VPC_ID" \
  --query "GroupId" \
  --output text \
  --region us-east-1)

echo $RDS_SG_ID
```

**Terminal Output:**

```bash
~ on ☁️  (us-east-1) ➜  RDS_SG_ID=$(aws ec2 create-security-group \
  --group-name $SECURITY_GROUP \
  --description "RDS SG" \
  --vpc-id "$VPC_ID" \
  --query "GroupId" \
  --output text \
  --region us-east-1)

echo $RDS_SG_ID
sg-0c472975a9422f2ea

~ on ☁️  (us-east-1) ➜
```

Security Group ID `sg-0c472975a9422f2ea` was created and stored in the `$RDS_SG_ID` variable.

***

### Phase 5: Create the RDS Instance

With all prerequisites in place, the RDS instance is created using the following configuration. Key parameters are explained in the table below:

| Parameter        | Flag                        | Value         | Purpose             |
| ---------------- | --------------------------- | ------------- | ------------------- |
| Instance name    | `--db-instance-identifier`  | `devops-rds`  | Unique identifier   |
| Instance class   | `--db-instance-class`       | `db.t3.micro` | Free tier eligible  |
| Engine           | `--engine`                  | `mysql`       | Database engine     |
| Engine version   | `--engine-version`          | `8.4.3`       | MySQL 8.4.x series  |
| Initial storage  | `--allocated-storage`       | `20` GB       | Starting disk size  |
| Max storage      | `--max-allocated-storage`   | `50` GB       | Autoscaling ceiling |
| Private access   | `--no-publicly-accessible`  | —             | No public endpoint  |
| Backup retention | `--backup-retention-period` | `1` day       | Minimum backup      |
| Storage type     | `--storage-type`            | `gp2`         | General Purpose SSD |

```bash
aws rds create-db-instance \
  --db-instance-identifier $RDS_NAME \
  --db-instance-class $INSTANCE_TYPE \
  --engine $ENGINE \
  --engine-version $ENGINE_VERSION \
  --allocated-storage 20 \
  --max-allocated-storage $MAX_STORAGE \
  --master-username admin \
  --master-user-password Admin1234! \
  --db-subnet-group-name $SUBNET_GROUP \
  --vpc-security-group-ids "$RDS_SG_ID" \
  --no-publicly-accessible \
  --backup-retention-period 1 \
  --storage-type gp2 \
  --region us-east-1
```

**Terminal Output:**

```bash
~ on ☁️  (us-east-1) ➜  aws rds create-db-instance \
  --db-instance-identifier $RDS_NAME \
  --db-instance-class $INSTANCE_TYPE \
  --engine $ENGINE \
  --engine-version $ENGINE_VERSION \
  --allocated-storage 20 \
  --max-allocated-storage $MAX_STORAGE \
  --master-username admin \
  --master-user-password Admin1234! \
  --db-subnet-group-name $SUBNET_GROUP \
  --vpc-security-group-ids "$RDS_SG_ID" \
  --no-publicly-accessible \
  --backup-retention-period 1 \
  --storage-type gp2 \
  --region us-east-1
{
    "DBInstance": {
        "DBInstanceIdentifier": "devops-rds",
        "DBInstanceClass": "db.t3.micro",
        "Engine": "mysql",
        "DBInstanceStatus": "creating",
        "MasterUsername": "admin",
        "AllocatedStorage": 20,
        "PreferredBackupWindow": "08:43-09:13",
        "BackupRetentionPeriod": 1,
        "DBSecurityGroups": [],
        "VpcSecurityGroups": [
            {
                "VpcSecurityGroupId": "sg-0c472975a9422f2ea",
                "Status": "active"
            }
        ],
        "DBParameterGroups": [
            {
                "DBParameterGroupName": "default.mysql8.4",
                "ParameterApplyStatus": "in-sync"
            }
        ],
        "DBSubnetGroup": {
            "DBSubnetGroupName": "db-subnet-group",
            "DBSubnetGroupDescription": "Subnet group for RDS",
            "VpcId": "vpc-0b73f32d24d9f9625",
            "SubnetGroupStatus": "Complete",
            "Subnets": [
                {
                    "SubnetIdentifier": "subnet-0968e1f342f7d5e00",
                    "SubnetAvailabilityZone": { "Name": "us-east-1e" },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0f1f9b4afc216545c",
                    "SubnetAvailabilityZone": { "Name": "us-east-1f" },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0480cba21faebe63f",
                    "SubnetAvailabilityZone": { "Name": "us-east-1a" },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0f42654d7f527bbc2",
                    "SubnetAvailabilityZone": { "Name": "us-east-1c" },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0637f846e34e2218e",
                    "SubnetAvailabilityZone": { "Name": "us-east-1b" },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-08f5b39e48ba03c99",
                    "SubnetAvailabilityZone": { "Name": "us-east-1d" },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                }
            ]
        },
        "PreferredMaintenanceWindow": "tue:06:15-tue:06:45",
        "PendingModifiedValues": {
            "MasterUserPassword": "****"
        },
        "MultiAZ": false,
        "EngineVersion": "8.4.3",
        "AutoMinorVersionUpgrade": true,
        "ReadReplicaDBInstanceIdentifiers": [],
        "LicenseModel": "general-public-license",
        "OptionGroupMemberships": [
            {
                "OptionGroupName": "default:mysql-8-4",
                "Status": "in-sync"
            }
        ],
        "PubliclyAccessible": false,
        "StorageType": "gp2",
        "DbInstancePort": 0,
        "StorageEncrypted": false,
        "DbiResourceId": "db-4ZSFL2LGV5E5S6XXS6OMJSEB24",
        "CACertificateIdentifier": "rds-ca-rsa2048-g1",
        "DomainMemberships": [],
        "CopyTagsToSnapshot": false,
        "MonitoringInterval": 0,
        "DBInstanceArn": "arn:aws:rds:us-east-1:467239208986:db:devops-rds",
        "IAMDatabaseAuthenticationEnabled": false,
        "DatabaseInsightsMode": "standard",
        "PerformanceInsightsEnabled": false,
        "DeletionProtection": false,
        "AssociatedRoles": [],
        "MaxAllocatedStorage": 50,
        "TagList": [],
        "CustomerOwnedIpEnabled": false,
        "BackupTarget": "region",
        "NetworkType": "IPV4",
        "StorageThroughput": 0,
        "CertificateDetails": {
            "CAIdentifier": "rds-ca-rsa2048-g1"
        },
        "DedicatedLogVolume": false,
        "EngineLifecycleSupport": "open-source-rds-extended-support"
    }
}

~ on ☁️  (us-east-1) ➜
```

The instance was accepted by AWS with `DBInstanceStatus: creating`. Key fields confirmed in the response:

* `PubliclyAccessible: false` — instance is private
* `MaxAllocatedStorage: 50` — autoscaling enabled with 50GB ceiling
* `EngineVersion: 8.4.3` — correct MySQL version
* `MultiAZ: false` — single AZ deployment (sandbox/free tier)
* `DBInstanceArn: arn:aws:rds:us-east-1:467239208986:db:devops-rds`

***

### Phase 6: Wait for Instance to Become Available

The `aws rds wait` command polls the instance status every 30 seconds and returns only when the status transitions to `available`. This typically takes 5–10 minutes.

```bash
aws rds wait db-instance-available \
  --db-instance-identifier $RDS_NAME \
  --region us-east-1
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  aws rds wait db-instance-available \
  --db-instance-identifier $RDS_NAME \
  --region us-east-1

~ on ☁️  (us-east-1) ➜
```

The command returned with no output, which indicates success. The instance is now in the `available` state.

***

### Phase 7: Validate the RDS Instance

Run a final describe command to confirm all configuration values match the lab requirements:

```bash
aws rds describe-db-instances \
  --db-instance-identifier $RDS_NAME \
  --query "DBInstances[0].[DBInstanceStatus,PubliclyAccessible,EngineVersion,DBInstanceClass,MaxAllocatedStorage]" \
  --output table \
  --region us-east-1
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  aws rds describe-db-instances \
  --db-instance-identifier $RDS_NAME \
  --query "DBInstances[0].[DBInstanceStatus,PubliclyAccessible,EngineVersion,DBInstanceClass,MaxAllocatedStorage]" \
  --output table \
  --region us-east-1
---------------------
|DescribeDBInstances|
+-------------------+
|  available        |
|  False            |
|  8.4.3            |
|  db.t3.micro      |
|  50               |
+-------------------+

~ on ☁️  (us-east-1) ➜
```

All five fields confirm the instance is correctly configured and ready for use.

***

### Summary

All lab objectives were completed successfully:

| Requirement           | Configured Value  | Status    |
| --------------------- | ----------------- | --------- |
| Instance identifier   | `devops-rds`      | Confirmed |
| Instance class        | `db.t3.micro`     | Confirmed |
| Database engine       | `MySQL`           | Confirmed |
| Engine version        | `8.4.3`           | Confirmed |
| Publicly accessible   | `False` (private) | Confirmed |
| Storage autoscaling   | Enabled           | Confirmed |
| Max storage threshold | `50 GB`           | Confirmed |
| Instance status       | `available`       | Confirmed |
| Region                | `us-east-1`       | Confirmed |

***

### Key Concepts

#### Why a DB Subnet Group is Required

Amazon RDS does not deploy directly into a VPC — it requires a **DB Subnet Group**, which is a named collection of subnets that RDS can choose from when placing the instance. The subnet group must span at least two Availability Zones to support Multi-AZ deployments (even when Multi-AZ is disabled, AWS enforces this requirement during creation).

#### Storage Autoscaling vs Allocated Storage

| Setting               | Flag                      | Description                             |
| --------------------- | ------------------------- | --------------------------------------- |
| `AllocatedStorage`    | `--allocated-storage`     | The initial provisioned disk size in GB |
| `MaxAllocatedStorage` | `--max-allocated-storage` | The upper ceiling for automatic scaling |

When `MaxAllocatedStorage` is set, RDS monitors free storage space and automatically expands the volume when it falls below 10% — up to the defined maximum. Setting this to `50` enables autoscaling with a 50GB ceiling as required.

#### Private vs Public RDS Instances

| Setting | Flag                       | Behaviour                                   |
| ------- | -------------------------- | ------------------------------------------- |
| Private | `--no-publicly-accessible` | Instance only reachable from within the VPC |
| Public  | `--publicly-accessible`    | AWS assigns a public DNS endpoint           |

For security best practices, production and development database instances should always be private. Access from application servers is handled through the VPC's internal network using the instance's private DNS endpoint.

#### The `aws rds wait` Command

The `wait` subcommand is a built-in AWS CLI polling mechanism that repeatedly calls `describe-db-instances` every 30 seconds and exits with code `0` when the target condition (`db-instance-available`) is met. It exits with code `255` if the condition is not met within the maximum wait time (approximately 30 minutes). This is preferable to manual polling in scripts and automation pipelines.

#### RDS Instance Status Lifecycle

During provisioning, an RDS instance transitions through the following states:

```
creating  -->  backing-up  -->  available
```

The instance is only ready for connections once it reaches the `available` state.

***

### Resource Reference

| Resource        | Type                   | Identifier                                         |
| --------------- | ---------------------- | -------------------------------------------------- |
| VPC             | Default VPC            | `vpc-0b73f32d24d9f9625`                            |
| Subnets         | 6 subnets across 6 AZs | `subnet-0968e1f342f7d5e00` ...                     |
| DB Subnet Group | RDS Subnet Group       | `db-subnet-group`                                  |
| Security Group  | EC2 Security Group     | `sg-0c472975a9422f2ea`                             |
| RDS Instance    | MySQL 8.4.3            | `devops-rds`                                       |
| Instance ARN    | AWS Resource Name      | `arn:aws:rds:us-east-1:467239208986:db:devops-rds` |
| Resource ID     | DBI Resource           | `db-4ZSFL2LGV5E5S6XXS6OMJSEB24`                    |
| Parameter Group | MySQL Config           | `default.mysql8.4`                                 |
| Option Group    | MySQL Options          | `default:mysql-8-4`                                |
| CA Certificate  | TLS Certificate        | `rds-ca-rsa2048-g1`                                |

***

_Lab completed on 2026-03-24 | AWS Region: us-east-1 | Platform: KodeKloud_

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

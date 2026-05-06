# Day 5: Create GP3 Volume

The Nautilus DevOps team is migrating infrastructure to AWS in incremental steps. For this task, create an EBS volume with the following requirements:

| Requirement | Value            |
| ----------- | ---------------- |
| Volume Name | `xfusion-volume` |
| Volume Type | `gp3`            |
| Volume Size | `2 GiB`          |
| Region      | `us-east-1`      |

**Notes:**

* Create the resources only in `us-east-1` region.

---

## AWS CLI Steps

### Step 1: Load AWS Credentials

On the **aws-client host**, run:

```bash
showcreds
```

This command exports temporary AWS credentials (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`) into your environment.

### Step 2: Set AWS Region

```bash
aws configure set region us-east-1
```

Verify:

```bash
aws configure get region
```

Expected output:

```
us-east-1
```

### Step 3: Create the EBS Volume

```bash
aws ec2 create-volume \
  --volume-type gp3 \
  --size 2 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=xfusion-volume}]'
```

**Notes:**

* Volume size is in **GiB** -> `--size 2`
* `gp3` is explicitly specified
* Availability Zone must belong to `us-east-1` (e.g., `us-east-1a`)
* Volume name is applied via **tags**

### Step 4: Verify the Volume

```bash
aws ec2 describe-volumes \
  --filters Name=tag:Name,Values=xfusion-volume \
  --query "Volumes[*].[VolumeId,Size,VolumeType,AvailabilityZone,State]" \
  --output table
```

Expected output:

```
-----------------------------------------------
|             DescribeVolumes                 |
+--------------+------+-------+---------+-----+
| vol-0abcd123 |  2   | gp3   | us-east-1a | available |
+--------------+------+-------+---------+-----+
```

---

## Result

* Volume name: `xfusion-volume`
* Volume type: `gp3`
* Size: `2 GiB`
* Region: `us-east-1`
* Status: `available`

---

```bash
aws ec2 create-volume \
  --volume-type gp3 \
  --size 2 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=xfusion-volume}]'
{
    "Iops": 3000,
    "Tags": [
        {
            "Key": "Name",
            "Value": "xfusion-volume"
        }
    ],
    "VolumeType": "gp3",
    "MultiAttachEnabled": false,
    "Throughput": 125,
    "VolumeId": "vol-02b73ec5973bc1d47",
    "Size": 2,
    "SnapshotId": "",
    "AvailabilityZone": "us-east-1a",
    "State": "creating",
    "CreateTime": "2026-01-10T15:25:40.000Z",
    "Encrypted": false
}

aws ec2 describe-volumes \
  --filters Name=tag:Name,Values=xfusion-volume \
  --query "Volumes[*].[VolumeId,Size,VolumeType,AvailabilityZone,State]" \
  --output table
------------------------------------------------------------------
|                         DescribeVolumes                        |
+------------------------+----+------+-------------+-------------+
|  vol-02b73ec5973bc1d47 |  2 |  gp3 |  us-east-1a |  available  |
+------------------------+----+------+-------------+-------------+
```

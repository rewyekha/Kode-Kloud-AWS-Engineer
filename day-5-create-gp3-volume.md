# Day 5: Create GP3 Volume

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

Create a volume with the following requirements:

* Name of the volume should be `xfusion-volume`.
* Volume `type` must be `gp3`.
* Volume `size` must be `2 GiB`.

**Notes:**

* Create the resources only in `us-east-1` region.

Below is the **clean, exam-style solution** to create the required **AWS EBS volume** using the **AWS CLI** from the `aws-client` host.

***

### Task Summary

You need to create an **EBS volume** with the following specifications:

| Requirement | Value                      |
| ----------- | -------------------------- |
| Volume Name | `xfusion-volume`           |
| Volume Type | `gp3`                      |
| Volume Size | `2 GiB`                    |
| Region      | `us-east-1`                |
| Credentials | Provided (via `showcreds`) |

***

### Step 1: Load AWS Credentials

On the **aws-client host**, run:

```bash
showcreds
```

This command exports temporary AWS credentials (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`) into your environment.

 **Do not hardcode credentials manually**

***

### Step 2: Set AWS Region

Ensure the region is set correctly:

```bash
aws configure set region us-east-1
```

(Optional check)

```bash
aws configure get region
```

Expected output:

```
us-east-1
```

***

### Step 3: Create the EBS Volume

Run the following command:

```bash
aws ec2 create-volume \
  --volume-type gp3 \
  --size 2 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=xfusion-volume}]'
```

 **Important Notes**

* Volume size is in **GiB** → `--size 2`
* `gp3` is explicitly specified
* Availability Zone must belong to `us-east-1` (e.g., `us-east-1a`)
* Volume name is applied via **tags**

***

### Step 4: Verify the Volume

```bash
aws ec2 describe-volumes \
  --filters Name=tag:Name,Values=xfusion-volume \
  --query "Volumes[*].[VolumeId,Size,VolumeType,AvailabilityZone,State]" \
  --output table
```

Expected output (example):

```
-----------------------------------------------
|             DescribeVolumes                 |
+--------------+------+-------+---------+-----+
| vol-0abcd123 |  2   | gp3   | us-east-1a | available |
+--------------+------+-------+---------+-----+
```

***

### Final Validation Checklist

* Volume name: `xfusion-volume`
* Volume type: `gp3`
* Size: `2 GiB`
* Region: `us-east-1`
* Status: `available`

***

### Conclusion

The **xfusion-volume** EBS resource has been successfully created following AWS best practices and regional constraints. This aligns perfectly with the Nautilus DevOps team’s **incremental migration strategy**, ensuring controlled rollout and reduced risk.

```

~ on ☁️  (us-east-1) ➜  aws ec2 create-volume \
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

~ on ☁️  (us-east-1) ➜  aws ec2 describe-volumes \
  --filters Name=tag:Name,Values=xfusion-volume \
  --query "Volumes[*].[VolumeId,Size,VolumeType,AvailabilityZone,State]" \
  --output table
------------------------------------------------------------------
|                         DescribeVolumes                        |
+------------------------+----+------+-------------+-------------+
|  vol-02b73ec5973bc1d47 |  2 |  gp3 |  us-east-1a |  available  |
+------------------------+----+------+-------------+-------------+

~ on ☁️  (us-east-1) ➜

```

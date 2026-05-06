# Day 13: Create AMI from EC2 Instance

Create an AMI from an existing EC2 instance named `xfusion-ec2` with the following requirement:

* The AMI name must be `xfusion-ec2-ami` and must be in `available` state.

**Notes:**

* Create the resources only in `us-east-1` region.

---

This guide describes the step-by-step process of creating an Amazon Machine Image (AMI) from an existing EC2 instance named `xfusion-ec2` using the AWS CLI. It includes actual command outputs and highlights a minor query formatting issue and its resolution.

## Prerequisites

* AWS CLI configured with appropriate credentials.
* Target EC2 instance exists with the tag `Name=xfusion-ec2`.
* Region set to `us-east-1`.

## Solution

### Step 1: Retrieve EC2 Instance ID by Tag

Run the following command to find the instance ID of the EC2 instance tagged as `xfusion-ec2`:

```bash
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=xfusion-ec2" \
    --query "Reservations[].Instances[].InstanceId" \
    --output table
```

Output:

```
-------------------------
|   DescribeInstances   |
+-----------------------+
|  i-0d408098b1c5ebf80  |
+-----------------------+
```

The instance ID is `i-0d408098b1c5ebf80`.

### Step 2: Create AMI from the Instance

Create the AMI named `xfusion-ec2-ami` from the instance with no reboot:

```bash
aws ec2 create-image \
    --instance-id i-0d408098b1c5ebf80 \
    --name "xfusion-ec2-ami" \
    --description "AMI for xfusion-ec2 migration" \
    --no-reboot
```

Output:

```json
{
    "ImageId": "ami-08f86cc56f14a407f"
}
```

AMI creation started with Image ID `ami-08f86cc56f14a407f`.

### Step 3: Check AMI State (Initial Attempts)

Query the AMI state:

```bash
aws ec2 describe-images \
    --image-ids ami-08f86cc56f14a407f \
    --query "Images[0].State" \
    --output table
```

Output (repeated multiple times):

```
----------------
|DescribeImages|
+--------------+
```

**Issue:** The output did not show the expected AMI state, only the header. This indicates a minor AWS CLI or query formatting issue.

### Step 4: Testing Invalid AMI ID

To confirm error handling, a wrong AMI ID was queried:

```bash
aws ec2 describe-images \
    --image-ids ami-0f123456789abcdef \
    --query "Images[0].State" \
    --output table
```

Output:

```
An error occurred (InvalidAMIID.NotFound) when calling the DescribeImages operation: The image id '[ami-0f123456789abcdef]' does not exist
```

### Step 5: Successful Query for AMI State (Using Text Output)

Switching the output format to `text` fixed the problem:

```bash
aws ec2 describe-images \
  --image-ids ami-08f86cc56f14a407f \
  --query "Images[0].State" \
  --output text
```

Output:

```
available
```

The AMI state is confirmed as `available`.

### Step 6: Inspect Full AMI Details

To verify all AMI metadata, retrieve the full JSON output:

```bash
aws ec2 describe-images --image-ids ami-08f86cc56f14a407f --output json
```

Output:

```json
{
    "Images": [
        {
            "PlatformDetails": "Linux/UNIX",
            "UsageOperation": "RunInstances",
            "BlockDeviceMappings": [
                {
                    "Ebs": {
                        "DeleteOnTermination": true,
                        "Iops": 3000,
                        "SnapshotId": "snap-0828bd9ecf9b6b31c",
                        "VolumeSize": 8,
                        "VolumeType": "gp3",
                        "Throughput": 125,
                        "Encrypted": false
                    },
                    "DeviceName": "/dev/xvda"
                }
            ],
            "Description": "AMI for xfusion-ec2 migration",
            "EnaSupport": true,
            "Hypervisor": "xen",
            "Name": "xfusion-ec2-ami",
            "RootDeviceName": "/dev/xvda",
            "RootDeviceType": "ebs",
            "SriovNetSupport": "simple",
            "VirtualizationType": "hvm",
            "BootMode": "uefi-preferred",
            "ImdsSupport": "v2.0",
            "SourceInstanceId": "i-0d408098b1c5ebf80",
            "DeregistrationProtection": "disabled",
            "SourceImageId": "ami-0c101f26f147fa7fd",
            "SourceImageRegion": "us-east-1",
            "ImageId": "ami-08f86cc56f14a407f",
            "ImageLocation": "062665041731/xfusion-ec2-ami",
            "State": "available",
            "OwnerId": "062665041731",
            "CreationDate": "2026-02-24T06:46:54.000Z",
            "Public": false,
            "Architecture": "x86_64",
            "ImageType": "machine"
        }
    ]
}
```

---

## Conclusion

* The AMI `xfusion-ec2-ami` was created successfully from EC2 instance `i-0d408098b1c5ebf80`.
* The AMI reached the `available` state as confirmed via CLI and AWS Console.
* Minor issues with the `--query` output formatting were resolved by switching the output format to `text`.
* Full metadata confirms the AMI details and configuration.

The AWS Console shows the AMI `xfusion-ec2-ami` with status **Available**, matching the CLI output.

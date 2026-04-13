# Day 13: Create AMI from EC2 Instance

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create an AMI from an existing EC2 instance named `xfusion-ec2` with the following requirement:

* Name of the AMI should be `xfusion-ec2-ami`, make sure AMI is in `available` state.

Use below given AWS Credentials: (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

| Console URL | [https://062665041731.signin.aws.amazon.com/console?region=us-east-1](https://062665041731.signin.aws.amazon.com/console?region=us-east-1) |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Username    | kk\_labs\_user\_570106                                                                                                                     |
| Password    | PKHqFdydT09G                                                                                                                               |
| Start Time  | Tue Feb 24 06:42:14 UTC 2026                                                                                                               |
| End Time    | Tue Feb 24 07:42:14 UTC 2026                                                                                                               |

\
`Notes:`

* Create the resources only in `us-east-1` region.
* To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:\
  ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)



```abap
This documentation describes the step-by-step process of creating an Amazon Machine Image (AMI) from an existing EC2 instance named `xfusion-ec2` using AWS CLI and includes the outputs encountered during the process. It also highlights a minor query issue and its resolution.

---
```

````bash
# Creating an AMI from an Existing EC2 Instance (xfusion-ec2)

## Prerequisites

- AWS CLI configured with appropriate credentials.
- Target EC2 instance exists with the tag `Name=xfusion-ec2`.
- Region set to `us-east-1`.
- AWS Console URL: https://062665041731.signin.aws.amazon.com/console?region=us-east-1
- Credentials:
  - Username: `kk_labs_user_570106`
  - Password: `PKHqFdydT09G`

---

## Step 1: Retrieve EC2 Instance ID by Tag

Run the following command to find the instance ID of the EC2 instance tagged as `xfusion-ec2`:

```bash
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=xfusion-ec2" \
    --query "Reservations[].Instances[].InstanceId" \
    --output table
````

#### Output:

```
-------------------------
|   DescribeInstances   |
+-----------------------+
|  i-0d408098b1c5ebf80  |
+-----------------------+
```

The instance ID is `i-0d408098b1c5ebf80`.

***

### Step 2: Create AMI from the Instance

Create the AMI named `xfusion-ec2-ami` from the instance with no reboot:

```bash
aws ec2 create-image \
    --instance-id i-0d408098b1c5ebf80 \
    --name "xfusion-ec2-ami" \
    --description "AMI for xfusion-ec2 migration" \
    --no-reboot
```

#### Output:

```json
{
    "ImageId": "ami-08f86cc56f14a407f"
}
```

AMI creation started with Image ID `ami-08f86cc56f14a407f`.

***

### Step 3: Check AMI State (Initial Attempts)

Attempt to query the AMI state with:

```bash
aws ec2 describe-images \
    --image-ids ami-08f86cc56f14a407f \
    --query "Images[0].State" \
    --output table
```

#### Output (repeated multiple times):

```
----------------
|DescribeImages|
+--------------+
```

**Issue:** The output did not show the expected AMI state, only the header. This indicates a minor AWS CLI or query formatting issue.

***

### Step 4: Testing Invalid AMI ID

To confirm error handling, a wrong AMI ID was queried:

```bash
aws ec2 describe-images \
    --image-ids ami-0f123456789abcdef \
    --query "Images[0].State" \
    --output table
```

#### Output:

```
An error occurred (InvalidAMIID.NotFound) when calling the DescribeImages operation: The image id '[ami-0f123456789abcdef]' does not exist
```

***

### Step 5: Successful Query for AMI State (Using Text Output)

Switching the output format to `text` fixed the problem:

```bash
aws ec2 describe-images \
  --image-ids ami-08f86cc56f14a407f \
  --query "Images[0].State" \
  --output text
```

#### Output:

```
available
```

The AMI state is confirmed as `available`.

***

### Step 6: Inspect Full AMI Details (JSON Output)

To verify all AMI metadata, the full JSON output was retrieved:

```bash
aws ec2 describe-images --image-ids ami-08f86cc56f14a407f --output json
```

#### Output:

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

***

### Conclusion

* The AMI `xfusion-ec2-ami` was created successfully from EC2 instance `i-0d408098b1c5ebf80`.
* The AMI reached the `available` state as confirmed via CLI and AWS Console.
* Minor issues with the `--query` output formatting were resolved by switching the output format to `text`.
* Full metadata confirms the AMI details and configuration.

***

### GUI Confirmation

The AWS Console shows the AMI `xfusion-ec2-ami` with status **Available**, matching the CLI outputs.

***

This concludes the AMI creation and verification process for the `xfusion-ec2` instance in `us-east-1`.

```
# End of Document
```

# Day 50: Expanding EC2 Instance Storage for Development Needs

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

The Nautilus DevOps Team has recently been informed by the Development Team that their EC2 instance is running out of storage space. This instance, crucial for development activities, is named `datacenter-ec2` and currently has an attached volume of `8 GiB`. To accommodate the increasing data requirements, the storage needs to be expanded to `12 GiB`. This change should ensure that the expanded space is immediately available for use within the instance without disrupting ongoing activities.

1. Identify Volume: Find the volume attached to the `datacenter-ec2` instance.
2. Expand Volume: Increase the volume size from `8 GiB` to `12 GiB`.
3. Reflect Changes: Ensure the root (`/`) partition within the instance reflects the expanded size from `8 GiB` to `12 GiB`.
4. SSH Access: Use the key pair located at `/root/datacenter-keypair.pem` on the `aws-client` host to SSH into the EC2 instance.

`Notes:`

* Create the resources only in `us-east-1` region.

## Expanding EBS Volume Storage on a Live EC2 Instance

---

### Overview

The Nautilus DevOps team was informed by the Development Team that their EC2 instance (`datacenter-ec2`) was running out of storage space. The attached EBS root volume of **8 GiB** needed to be expanded to **12 GiB** — and the expanded space had to be immediately available inside the instance without any downtime or service disruption.

All resources are in the **us-east-1 (N. Virginia)** AWS region.

---

### Objectives

* Identify the EBS volume attached to `datacenter-ec2`
* Expand the volume from **8 GiB → 12 GiB** using the AWS CLI
* Extend the partition to use the new space using `growpart`
* Grow the live XFS filesystem using `xfs_growfs` without unmounting
* Verify the expanded space is reflected inside the instance

---

### Prerequisites

* AWS CLI configured on the `aws-client` host
* SSH key pair available at `/root/datacenter-keypair.pem`
* EC2 instance `datacenter-ec2` running and accessible
* Permissions to modify EBS volumes and SSH into the instance

---

### Architecture Overview

```bash
┌─────────────────────────────────────────────┐
│           datacenter-ec2 (EC2 Instance)      │
│           Amazon Linux 2023                  │
│           IP: 54.160.245.198                 │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │  EBS Volume: vol-02b8e149d982b8fa7     │  │
│  │  Type: gp3                             │  │
│  │  Before: 8 GiB  →  After: 12 GiB      │  │
│  │                                        │  │
│  │  /dev/xvda  (disk)                     │  │
│  │  └─ /dev/xvda1   →  / (root)          │  │
│  │  └─ /dev/xvda128 →  /boot/efi         │  │
│  └────────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

---

### Step-by-Step Walkthrough

#### Step 1 — Identify the EC2 Instance

Retrieve the instance ID of `datacenter-ec2` using a tag filter:

```bash
INSTANCE_ID=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=datacenter-ec2" \
  --query "Reservations[0].Instances[0].InstanceId" \
  --output text)

echo $INSTANCE_ID
```

**Output:**

```
i-05836b492f7edcd1f
```

---

#### Step 2 — Identify the Attached EBS Volume

Retrieve the volume ID from the instance's block device mappings:

```bash
VOLUME_ID=$(aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].BlockDeviceMappings[0].Ebs.VolumeId" \
  --output text)

echo $VOLUME_ID
```

**Output:**

```
vol-02b8e149d982b8fa7
```

Confirm current volume size and state:

```bash
aws ec2 describe-volumes \
  --volume-ids $VOLUME_ID \
  --query "Volumes[].{Volume:VolumeId,Size:Size,State:State}" \
  --output table
```

**Output:**

```
---------------------------------------------
|              DescribeVolumes              |
+------+---------+--------------------------+
| Size |  State  |         Volume           |
+------+---------+--------------------------+
|  8   |  in-use |  vol-02b8e149d982b8fa7   |
+------+---------+--------------------------+
```

---

#### Step 3 — Expand the EBS Volume to 12 GiB

Modify the volume size from 8 GiB to 12 GiB using the AWS CLI:

```bash
aws ec2 modify-volume \
  --volume-id $VOLUME_ID \
  --size 12
```

**Output:**

```json
{
    "VolumeModification": {
        "VolumeId": "vol-02b8e149d982b8fa7",
        "ModificationState": "modifying",
        "TargetSize": 12,
        "TargetIops": 3000,
        "TargetVolumeType": "gp3",
        "TargetThroughput": 125,
        "OriginalSize": 8,
        "OriginalVolumeType": "gp3",
        "Progress": 0,
        "StartTime": "2026-05-03T01:22:17.000Z"
    }
}
```

Verify the volume now reflects 12 GiB:

```bash
aws ec2 describe-volumes \
  --volume-ids $VOLUME_ID \
  --query "Volumes[].{Volume:VolumeId,Size:Size,State:State}" \
  --output table
```

**Output:**

```
---------------------------------------------
|              DescribeVolumes              |
+------+---------+--------------------------+
| Size |  State  |         Volume           |
+------+---------+--------------------------+
|  12  |  in-use |  vol-02b8e149d982b8fa7   |
+------+---------+--------------------------+
```

>  At this point, AWS has expanded the underlying EBS volume. However, the OS inside the instance still sees the old 8 GiB partition. The next steps reflect this change inside the instance.

---

#### Step 4 — SSH into the EC2 Instance

Retrieve the public IP and connect via SSH:

```bash
PUBLIC_IP=$(aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[0].Instances[0].PublicIpAddress" \
  --output text)

ssh -i /root/datacenter-keypair.pem ec2-user@$PUBLIC_IP
```

**Output:**

```
Warning: Permanently added '54.160.245.198' (ECDSA) to the list of known hosts.

   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|       https://aws.amazon.com/linux/amazon-linux-2023
   ~~       \#/ ___
    ~~       V~' '->
     ~~~         /
       ~~._.   _/
          _/ _/
        _/m/'
```

---

#### Step 5 — Verify Block Device Layout

Check the current disk and partition layout before making any changes:

```bash
lsblk
```

**Output:**

```
NAME      MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
xvda      202:0    0  12G  0 disk
├─xvda1   202:1    0   8G  0 part /
├─xvda127 259:0    0   1M  0 part
└─xvda128 259:1    0  10M  0 part /boot/efi
```

> The disk (`xvda`) already shows **12G** because AWS expanded the volume. However, partition `xvda1` still shows only **8G** — this is what needs to be extended.

---

#### Step 6 — Extend the Partition with `growpart`

Use `growpart` to resize partition 1 to fill the available disk space:

```bash
sudo growpart /dev/xvda 1
```

**Output:**

```
CHANGED: partition=1 start=24576 old: size=16752607 end=16777183 new: size=25141215 end=25165791
```

---

#### Step 7 — Grow the XFS Filesystem

The root filesystem (`/`) uses XFS. Use `xfs_growfs` to expand it online without unmounting:

```bash
sudo xfs_growfs -d /
```

**Output:**

```
meta-data=/dev/xvda1             isize=512    agcount=2, agsize=1047040 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1
data     =                       bsize=4096   blocks=2094075, imaxpct=25
         =                       sunit=128    swidth=128 blks
naming   =version 2              bsize=16384  ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=4096  sunit=4 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2094075 to 3142651
```

> `data blocks changed from 2094075 to 3142651` confirms the filesystem was successfully grown.

---

#### Step 8 — Verify the Expanded Filesystem

Confirm the root partition now reflects 12 GiB:

```bash
df -h
```

**Output:**

```
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        4.0M     0  4.0M   0% /dev
tmpfs           475M     0  475M   0% /dev/shm
tmpfs           190M  2.9M  188M   2% /run
/dev/xvda1       12G  1.6G   11G  13% /
tmpfs           475M     0  475M   0% /tmp
/dev/xvda128     10M  1.3M  8.7M  13% /boot/efi
tmpfs            95M    0   95M    0% /run/user/1000
```

 `/dev/xvda1` now shows **12G** with **11G** available — the expanded storage is fully reflected in the running instance.

---

### Verification Summary

| Check                    | Before | After  | Status |
| ------------------------ | ------ | ------ | ------ |
| EBS volume size (AWS)    | 8 GiB  | 12 GiB |       |
| Partition size (`xvda1`) | 8G     | 12G    |       |
| Filesystem size (`/`)    | 8.0G   | 12G    |       |
| Available space          | 6.5G   | 11G    |       |
| Instance downtime        | —      | None   |       |

---

### How It Works — Concept Explained

Expanding an EBS volume on a live Linux instance requires **three separate steps**. Each layer must be explicitly expanded:

```bash
┌─────────────────────────────────────┐
│  1. AWS EBS Volume (aws ec2         │  ← Expands the raw block device
│     modify-volume)                  │     visible to the hypervisor
├─────────────────────────────────────┤
│  2. Partition Table (growpart)      │  ← Expands the partition boundary
│                                     │     inside the disk
├─────────────────────────────────────┤
│  3. Filesystem (xfs_growfs /        │  ← Expands the filesystem to fill
│     or resize2fs for ext4)          │     the enlarged partition
└─────────────────────────────────────┘
```

> **Filesystem command depends on the type:**
>
> * XFS (Amazon Linux 2023, AL2): `sudo xfs_growfs -d /`
> * ext4 (Ubuntu, Debian): `sudo resize2fs /dev/xvda1`

---

### Key Takeaways

* EBS volumes can be expanded **without stopping the instance** — AWS supports live volume modification
* Simply expanding the volume in AWS is **not enough** — the partition and filesystem must also be grown inside the OS
* `growpart` handles partition resizing; `xfs_growfs` handles filesystem resizing for XFS volumes
* Always run `lsblk` and `df -h` before and after to confirm each layer of the expansion
* The entire operation caused **zero downtime** for the running instance

---

### Key Commands Reference

| Command                           | Purpose                        |
| --------------------------------- | ------------------------------ |
| `aws ec2 modify-volume --size 12` | Expand EBS volume at AWS level |
| `lsblk`                           | View disk and partition layout |
| `sudo growpart /dev/xvda 1`       | Extend partition to fill disk  |
| `sudo xfs_growfs -d /`            | Grow XFS filesystem online     |
| `df -h`                           | Verify filesystem sizes        |

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

---

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

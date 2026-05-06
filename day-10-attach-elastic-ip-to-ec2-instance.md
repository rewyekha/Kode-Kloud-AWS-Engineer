# Day 10: Attach Elastic IP to EC2 Instance

Attach the existing Elastic IP `nautilus-ec2-eip` to the existing EC2 instance `nautilus-ec2` in the `us-east-1` region.

**Notes:**

* Create the resources only in `us-east-1` region.

## Concept

An **Elastic IP (EIP)** in Amazon EC2 is a static public IPv4 address designed for dynamic cloud computing. Attaching an EIP ensures the public IP does not change after instance restarts.

## Solution

### Step 1: Get EC2 Instance ID

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=nautilus-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text
```

Example output:

```
i-0abc1234def567890
```

### Step 2: Get Elastic IP Allocation ID

```bash
aws ec2 describe-addresses \
  --filters "Name=tag:Name,Values=nautilus-ec2-eip" \
  --query "Addresses[].AllocationId" \
  --output text
```

Example output:

```
eipalloc-0123456789abcdef0
```

### Step 3: Attach Elastic IP to EC2

```bash
aws ec2 associate-address \
  --instance-id i-0abc1234def567890 \
  --allocation-id eipalloc-0123456789abcdef0
```

Successful execution returns an **AssociationId**:

```json
{
    "AssociationId": "eipassoc-0045976d3b4818dfa"
}
```

### Step 4: Verify Attachment

```bash
aws ec2 describe-addresses \
  --allocation-ids eipalloc-0ac22d845d7da377b
```

Expected output includes `InstanceId` and `AssociationId` fields confirming the attachment.

---

## Troubleshooting

**Common mistake:** Passing an Association ID (`eipassoc-`) to `describe-addresses` instead of the Allocation ID (`eipalloc-`).

The `describe-addresses` command expects an **Allocation ID**:

```bash
# Correct — use Allocation ID
aws ec2 describe-addresses \
  --allocation-ids eipalloc-0ac22d845d7da377b

# Incorrect — Association ID will fail
aws ec2 describe-addresses \
  --allocation-ids eipassoc-0045976d3b4818dfa
```

Error returned when using an Association ID:

```
InvalidAllocationID.NotFound
```

### ID Types Reference

| ID Type | Prefix | Used For |
| --- | --- | --- |
| Allocation ID | `eipalloc-` | Attach / describe EIP |
| Association ID | `eipassoc-` | Confirms attachment |
| Instance ID | `i-` | EC2 instance |

`describe-addresses` always uses `eipalloc-*`.

---

## Verification Options

**Option 1: Verify using Allocation ID (recommended)**

```bash
aws ec2 describe-addresses \
  --allocation-ids eipalloc-0ac22d845d7da377b
```

Expected: `"InstanceId"` and `"AssociationId"` fields are populated.

**Option 2: Verify via EC2 instance public IP**

```bash
aws ec2 describe-instances \
  --instance-ids i-066be603cea44b979 \
  --query "Reservations[].Instances[].PublicIpAddress" \
  --output text
```

This returns the Elastic IP address assigned to the instance.

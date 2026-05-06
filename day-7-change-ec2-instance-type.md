# Day 7: Change EC2 Instance Type

During the migration process, the Nautilus DevOps team discovered that one EC2 instance was underutilized and decided to change its instance type. Ensure the `Status check` is completed before making any changes.

1. Change the instance type from `t2.micro` to `t2.nano` for the `datacenter-ec2` instance.
2. Make sure the EC2 instance `datacenter-ec2` is in `running` state after the change.

**Notes:**

* Create the resources only in `us-east-1` region.

---

## AWS CLI Steps

### Step 0: Load Credentials

```bash
ssh aws-client
showcreds
```

Export the credentials from `showcreds`:

```bash
export AWS_ACCESS_KEY_ID=XXXX
export AWS_SECRET_ACCESS_KEY=YYYY
export AWS_DEFAULT_REGION=us-east-1
```

### Step 1: Get Instance ID for `datacenter-ec2`

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=datacenter-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text
```

Save the output and assign it:

```bash
INSTANCE_ID=i-0abc12345def67890
```

### Step 2: Ensure Status Checks Are Complete

```bash
aws ec2 describe-instance-status \
  --instance-ids $INSTANCE_ID \
  --query "InstanceStatuses[].InstanceStatus.Status" \
  --output text
```

Proceed only if output is `ok`. If empty or `initializing`, wait 1-2 minutes and retry.

### Step 3: Stop the Instance

Stopping is required to change the instance type.

```bash
aws ec2 stop-instances --instance-ids $INSTANCE_ID
```

Wait until stopped:

```bash
aws ec2 wait instance-stopped --instance-ids $INSTANCE_ID
```

### Step 4: Change Instance Type to `t2.nano`

```bash
aws ec2 modify-instance-attribute \
  --instance-id $INSTANCE_ID \
  --instance-type "{\"Value\": \"t2.nano\"}"
```

### Step 5: Start the Instance

```bash
aws ec2 start-instances --instance-ids $INSTANCE_ID
```

Wait until running:

```bash
aws ec2 wait instance-running --instance-ids $INSTANCE_ID
```

### Step 6: Verify

#### Check instance state

```bash
aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[].Instances[].State.Name" \
  --output text
```

Expected:

```
running
```

#### Check instance type

```bash
aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query "Reservations[].Instances[].InstanceType" \
  --output text
```

Expected:

```
t2.nano
```

---

## Result

* Correct instance identified: `datacenter-ec2`
* Status checks verified before modification
* Instance stopped, type changed to `t2.nano`, and restarted
* Region: `us-east-1`

> **Note:** You cannot change an EC2 instance type while it is running. Stopping -> modifying -> starting is mandatory.

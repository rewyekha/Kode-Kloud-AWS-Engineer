# Day 7: Change EC2 Instance Type

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

During the migration process, the Nautilus DevOps team created several EC2 instances in different regions. They are currently in the process of identifying the correct resources and utilization and are making continuous changes to ensure optimal resource utilization. Recently, they discovered that one of the EC2 instances was underutilized, prompting them to decide to change the instance type. Please make sure the `Status check` is completed (if its still in `Initializing` state) before making any changes to the instance.

1\) Change the instance type from `t2.micro` to `t2.nano` for `datacenter-ec2` instance.

2\) Make sure the ec2 instance `datacenter-ec2` is in `running` state after the change.

Use below given AWS Credentials: (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

| Console URL | https://<aws-account>.signin.aws.amazon.com/console?region=us-east-1 |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Username    | <REDACTED_USERNAME> |
| Password    | <REDACTED_PASSWORD> |
| Start Time  | Mon Jan 12 04:51:53 UTC 2026                                                                                                               |
| End Time    | Mon Jan 12 05:51:53 UTC 2026                                                                                                               |

`Notes:`

* Create the resources only in `us-east-1` region.
* Find EC2 instance **datacenter-ec2**
* Ensure **status checks are complete**
* Change instance type **t2.micro → t2.nano**
* Ensure instance is **running** afterward
* Region: **us-east-1**

---

###  Step 0: Login to aws-client & load credentials

```bash
ssh aws-client
showcreds
```

Export the credentials you get from `showcreds`, for example:

```bash
export AWS_ACCESS_KEY_ID=XXXX
export AWS_SECRET_ACCESS_KEY=YYYY
export AWS_DEFAULT_REGION=us-east-1
```

---

###  Step 1: Get Instance ID for `datacenter-ec2`

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=datacenter-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text
```

 Save the output (example):

```
i-0abc12345def67890
```

Assume:

```bash
INSTANCE_ID=i-0abc12345def67890
```

---

###  Step 2: Ensure Status Checks Are Complete

```bash
aws ec2 describe-instance-status \
  --instance-ids $INSTANCE_ID \
  --query "InstanceStatuses[].InstanceStatus.Status" \
  --output text
```

 Proceed **only if output is**:

```
ok
```

(If empty or `initializing`, wait 1–2 minutes and retry.)

---

###  Step 3: Stop the Instance (Required to change type)

```bash
aws ec2 stop-instances --instance-ids $INSTANCE_ID
```

Wait until stopped:

```bash
aws ec2 wait instance-stopped --instance-ids $INSTANCE_ID
```

---

###  Step 4: Change Instance Type to `t2.nano`

```bash
aws ec2 modify-instance-attribute \
  --instance-id $INSTANCE_ID \
  --instance-type "{\"Value\": \"t2.nano\"}"
```

---

###  Step 5: Start the Instance

```bash
aws ec2 start-instances --instance-ids $INSTANCE_ID
```

Wait until running:

```bash
aws ec2 wait instance-running --instance-ids $INSTANCE_ID
```

---

###  Step 6: Final Verification

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

###  Final Checklist (Exam Ready)

 Correct instance identified (`datacenter-ec2`)\
 Status checks verified\
 Instance stopped before modification\
 Instance type changed to `t2.nano`\
 Instance restarted and running\
 Region used: `us-east-1`

---

####  Exam Tip (Very Important)

> **You cannot change EC2 instance type while it is running**\
> Stopping → modifying → starting is **mandatory**

---

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

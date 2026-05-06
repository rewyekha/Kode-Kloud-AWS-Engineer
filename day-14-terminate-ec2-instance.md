# Day 14: Terminate EC2 Instance

An EC2 instance named `xfusion-ec2` in the `us-east-1` region is no longer in use and must be deleted.

1. Delete the EC2 instance named `xfusion-ec2` present in `us-east-1` region.
2. Before submitting your task, make sure the instance is in `terminated` state.

**Notes:**

* Create the resources only in `us-east-1` region.

<figure><img src=".gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

---

## Solution

### 1. Identify the EC2 Instance ID

Retrieve the **instance ID** of the EC2 instance named `xfusion-ec2`:

```bash
aws ec2 describe-instances \
    --region us-east-1 \
    --filters "Name=tag:Name,Values=xfusion-ec2" \
    --query "Reservations[].Instances[].InstanceId" \
    --output text
```

Expected output:

```
i-00de7cb1a11e41b66
```

> Note the instance ID — it will be used in the termination step.

**Common mistakes:**

* Using the wrong region (`us-west-2`, etc.) will return no results.
* A typo in the tag name (`xfusion-ec2`) will cause the instance not to be found.

### 2. Terminate the EC2 Instance

Use the retrieved instance ID to terminate the instance:

```bash
aws ec2 terminate-instances \
    --instance-ids i-00de7cb1a11e41b66 \
    --region us-east-1
```

Expected output:

```json
{
    "TerminatingInstances": [
        {
            "InstanceId": "i-00de7cb1a11e41b66",
            "CurrentState": {
                "Code": 48,
                "Name": "terminated"
            },
            "PreviousState": {
                "Code": 48,
                "Name": "terminated"
            }
        }
    ]
}
```

* `CurrentState: terminated` indicates that the instance is no longer running.

**Common mistakes:**

* Using an incorrect instance ID will cause termination to fail.
* Not specifying the correct region means AWS CLI cannot find the instance.

### 3. Verify Termination

Confirm that the EC2 instance is fully terminated:

```bash
aws ec2 describe-instances \
    --instance-ids i-00de7cb1a11e41b66 \
    --region us-east-1 \
    --query "Reservations[].Instances[].State.Name" \
    --output text
```

Expected output:

```
terminated
```

Once the output shows `terminated`, the instance has been successfully deleted.

**Tips:**

* If you see `shutting-down`, wait a few seconds and check again.
* Ensure that any dependent resources (EBS volumes, Elastic IPs) are deleted or detached if no longer needed.

---

## Summary

| Task | Detail |
| --- | --- |
| EC2 Instance | xfusion-ec2 |
| Region | us-east-1 |
| Action | Terminated using AWS CLI |
| Verification | Instance state is `terminated` |

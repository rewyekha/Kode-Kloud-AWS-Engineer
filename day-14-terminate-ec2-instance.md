# Day 14: Terminate EC2 Instance

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

During the migration process, several resources were created under the AWS account. Later on, some of these resources became obsolete as alternative solutions were implemented. Similarly, there is an instance that needs to be deleted as it is no longer in use.

1\) Delete the ec2 instance named `xfusion-ec2` present in `us-east-1` region.

2\) Before submitting your task, make sure instance is in `terminated` state.

Use below given AWS Credentials: (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

| Console URL | https://<aws-account>.signin.aws.amazon.com/console?region=us-east-1 |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Username    | <REDACTED_USERNAME> |
| Password    | <REDACTED_PASSWORD> |
| Start Time  | Wed Feb 25 05:07:55 UTC 2026                                                                                                               |
| End Time    | Wed Feb 25 06:07:55 UTC 2026                                                                                                               |

`Notes:`

* Create the resources only in `us-east-1` region.
* To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:\
  ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)

<figure><img src=".gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

---

During cloud resource management, some EC2 instances may become obsolete. This guide explains how to safely delete an EC2 instance (`xfusion-ec2`) in the **us-east-1** region using the AWS CLI and verify it is terminated.

---

### 1. Identify the EC2 Instance ID

First, retrieve the **instance ID** of the EC2 instance named `xfusion-ec2`:

```bash
aws ec2 describe-instances \
    --region us-east-1 \
    --filters "Name=tag:Name,Values=xfusion-ec2" \
    --query "Reservations[].Instances[].InstanceId" \
    --output text
```

**Expected Output:**

```
i-00de7cb1a11e41b66
```

> Note the instance ID, which will be used in the termination step.

**Common Mistakes:**

* Using the wrong **region** (`us-west-2`, etc.) will return no results.
* Typo in the tag name (`xfusion-ec2`) → Instance will not be found.

---

### 2. Terminate the EC2 Instance

Use the retrieved instance ID to terminate the instance:

```bash
aws ec2 terminate-instances \
    --instance-ids i-00de7cb1a11e41b66 \
    --region us-east-1
```

**Expected Output:**

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
* `PreviousState: terminated` confirms it was already shutting down or running before termination.

**Common Mistakes:**

* Using an **incorrect instance ID** → Termination will fail.
* Not specifying the correct **region** → AWS CLI cannot find the instance.

---

### 3. Verify Termination

Confirm that the EC2 instance is fully terminated:

```bash
aws ec2 describe-instances \
    --instance-ids i-00de7cb1a11e41b66 \
    --region us-east-1 \
    --query "Reservations[].Instances[].State.Name" \
    --output text
```

**Expected Output:**

```
terminated
```

* Once the output shows `terminated`, the instance has been successfully deleted.

**Tips:**

* If you see `shutting-down`, wait a few seconds and check again.
* Ensure that any dependent resources (EBS volumes, Elastic IPs) are deleted or detached if no longer needed.

---

###  Summary

| Task         | Detail                         |
| ------------ | ------------------------------ |
| EC2 Instance | xfusion-ec2                    |
| Region       | us-east-1                      |
| Action       | Terminated using AWS CLI       |
| Verification | Instance state is `terminated` |

**Status:**  Completed

---

---

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

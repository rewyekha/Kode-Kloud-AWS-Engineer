# Day 25: Setting Up an EC2 Instance and CloudWatch Alarm

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

## Setting Up EC2 Instance and CloudWatch Alarm for CPU Utilization Monitoring

### Overview

This guide describes how the Nautilus DevOps team set up an Amazon EC2 instance and configured a CloudWatch alarm to monitor CPU utilization. The alarm notifies an existing SNS topic when the CPU usage exceeds 90% for a consecutive 5-minute period.

---

### Objectives

* Launch an EC2 instance named **xfusion-ec2** using an Ubuntu AMI.
* Create a CloudWatch alarm named **xfusion-alarm** monitoring CPU utilization.
* Configure the alarm to trigger when CPU utilization exceeds 90% for 1 consecutive 5-minute period.
* Send alarm notifications to the existing SNS topic **xfusion-sns-topic**.

---

### Prerequisites

* AWS account access with credentials for `<REDACTED_USERNAME>`.
* Region set to `us-east-1`.
* Access to the existing SNS topic `xfusion-sns-topic`.

---

### Steps

#### 1. Launch EC2 Instance

* Navigate to the EC2 service in AWS Management Console.
* Click **Launch Instance**.
* Choose an appropriate **Ubuntu Server AMI**.
* Set the instance **Name tag** to `xfusion-ec2`.
* Select instance type (e.g., `t2.micro`).
* Use default VPC and subnet.
* Proceed to launch without a key pair (if SSH is not required).
* Confirm the instance state is **Running** after launch.

#### 2. Create CloudWatch Alarm

* Open the CloudWatch service in the AWS Console.
* Go to **Alarms** → **Create alarm**.
* Select metric:
  * Namespace: `AWS/EC2`
  * Metric: `CPUUtilization` for instance `xfusion-ec2`.
* Set the alarm conditions:
  * Statistic: **Average**
  * Period: **5 minutes**
  * Threshold: **Greater than or equal to 90%**
  * Evaluation period: **1 (consecutive 5-minute period)**
* For **Alarm actions**, choose the existing SNS topic **xfusion-sns-topic**.
* Name the alarm **xfusion-alarm**.
* Create the alarm.

---

### Verification

* Use AWS CLI or Console to check that:
  * The EC2 instance `xfusion-ec2` is running.
  * The CloudWatch alarm `xfusion-alarm` exists and monitors the correct metric with the specified threshold.
  * The alarm action is linked to `xfusion-sns-topic`.

---

### Notes

* The SNS topic `xfusion-sns-topic` was already created; no additional subscription management was required.
* The alarm may show **INSUFFICIENT\_DATA** initially until enough metric data is collected.
* To test the alarm, CPU load on the instance must exceed the threshold for at least 5 minutes.

---

### AWS CLI Commands Used for Verification

```bash
# Verify EC2 instance
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=xfusion-ec2" \
  --region us-east-1 \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name,Tags[?Key=='Name'].Value|[0]]" \
  --output table

# Verify CloudWatch alarm
aws cloudwatch describe-alarms \
  --alarm-names xfusion-alarm \
  --region us-east-1 \
  --query "MetricAlarms[*].[AlarmName,StateValue,Threshold,MetricName,Namespace,Period,EvaluationPeriods,AlarmActions]" \
  --output table
```

---

### Conclusion

The EC2 instance and CloudWatch alarm were successfully created according to the lab requirements. The alarm monitors CPU utilization and is configured to notify the existing SNS topic when CPU usage exceeds the specified threshold.

<figure><img src=".gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

---

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

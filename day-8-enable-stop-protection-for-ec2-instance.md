# Day 8: Enable Stop Protection for EC2 Instance





As part of the migration, there were some components added to the AWS account. Team created one of the EC2 instances where they need to make some changes now.

There is an EC2 instance named `datacenter-ec2` under `us-east-1` region, enable the `stop` protection for this instance.

Use below given AWS Credentials: (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

| Console URL | [https://415500486906.signin.aws.amazon.com/console?region=us-east-1](https://415500486906.signin.aws.amazon.com/console?region=us-east-1) |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Username    | kk\_labs\_user\_401572                                                                                                                     |
| Password    | 9k@xd7c!^^!D                                                                                                                               |
| Start Time  | Mon Jan 12 16:37:35 UTC 2026                                                                                                               |
| End Time    | Mon Jan 12 17:37:35 UTC 2026                                                                                                               |

\
`Notes:`

* Create the resources only in `us-east-1` region.



```bash
~ on ☁️  (us-east-1) ➜  aws ec2 describe-instances \
  --region us-east-1 \
  --filters "Name=tag:Name,Values=datacenter-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text
i-0dddbe6a69a39f4d0

~ on ☁️  (us-east-1) ➜  aws ec2 modify-instance-attribute \
  --region us-east-1 \
  --instance-id i-0dddbe6a69a39f4d0 \
  --disable-api-stop

~ on ☁️  (us-east-1) ➜  aws ec2 describe-instance-attribute \
  --region us-east-1 \
  --instance-id i-0dddbe6a69a39f4d0 \
  --attribute disableApiStop
{
    "InstanceId": "i-0dddbe6a69a39f4d0",
    "DisableApiStop": {
        "Value": true
    }
}

~ on ☁️  (us-east-1) ➜  

```



> Stop protection prevents the instance from being stopped via API/CLI/Console.

***

### 1. Configure AWS CLI (on aws-client host)

Run `showcreds` on the **aws-client** host and then configure AWS CLI:

```bash
aws configure
```

Provide:

* **AWS Access Key ID**: (from `showcreds`)
* **AWS Secret Access Key**: (from `showcreds`)
* **Default region name**: us-east-1
* **Default output format**: json

````

Verify:
```bash
aws sts get-caller-identity
````

***

### 2. Find the Instance ID for `datacenter-ec2`

```bash
aws ec2 describe-instances \
  --region us-east-1 \
  --filters "Name=tag:Name,Values=datacenter-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text
```

Example output:

```
i-0abc1234def567890
```

***

### 3. Enable Stop Protection on the Instance

Replace `<INSTANCE_ID>` with the value from the previous step.

```bash
aws ec2 modify-instance-attribute \
  --region us-east-1 \
  --instance-id <INSTANCE_ID> \
  --disable-api-stop
```

***

### 4. (Optional) Verify Stop Protection Is Enabled

```bash
aws ec2 describe-instance-attribute \
  --region us-east-1 \
  --instance-id <INSTANCE_ID> \
  --attribute disableApiStop
```

Expected output:

```json
{
  "DisableApiStop": {
    "Value": true
  }
}
```

***

#### ✅ Result

Stop protection is now enabled for **datacenter-ec2** in **us-east-1**.



### Enable Stop Protection via AWS Console (GUI)

1. **Sign in to AWS Console**
   * Open: [https://415500486906.signin.aws.amazon.com/console?region=us-east-1](https://415500486906.signin.aws.amazon.com/console?region=us-east-1)
   * Username: `kk_labs_user_401572`
   * Password: `9k@xd7c!^^!D`
2. **Go to EC2**
   * From the AWS Console home page, select **Services**
   * Click **EC2**
3. **Make sure you are in the correct region**
   * Top-right corner → select **US East (N. Virginia)**
4. **Open Instances**
   * In the left navigation pane, click **Instances**
5. **Select the instance**
   * Locate the instance named **`datacenter-ec2`**
   * Select the checkbox next to the instance
6. **Enable Stop Protection**
   * With the instance selected, click **Actions**
   * Choose **Instance settings**
   * Click **Change stop protection**
   * Check **Enable stop protection**
   * Click **Save**
7. **Confirmation**
   * You should see a confirmation message that stop protection has been enabled

***

#### ✅ Result

The EC2 instance **datacenter-ec2** is now protected from being stopped via the AWS Console, CLI, or API.

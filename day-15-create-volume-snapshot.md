# Day 15: Create Volume Snapshot

The Nautilus DevOps team has some volumes in different regions in their AWS account. They are going to setup some automated backups so that all important data can be backed up on regular basis. For now they shared some requirements to take a snapshot of one of the volumes they have.

Create a snapshot of an existing volume named `nautilus-vol` in `us-east-1` region.

1\) The name of the snapshot must be `nautilus-vol-ss`.

2\) The description must be `nautilus Snapshot`.

3\) Make sure the snapshot status is `completed` before submitting the task.

> **Note:** Run `showcreds` on the `aws-client` host to retrieve temporary AWS credentials, then configure the AWS CLI using `aws configure`.

**Notes:**

* Create the resources only in `us-east-1` region.
* To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:\
  ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
  <figure><img src=".gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>


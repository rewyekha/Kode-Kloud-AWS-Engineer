# Day 15: Create Volume Snapshot

The Nautilus DevOps team has some volumes in different regions in their AWS account. They are going to setup some automated backups so that all important data can be backed up on regular basis. For now they shared some requirements to take a snapshot of one of the volumes they have.

Create a snapshot of an existing volume named `nautilus-vol` in `us-east-1` region.

1\) The name of the snapshot must be `nautilus-vol-ss`.

2\) The description must be `nautilus Snapshot`.

3\) Make sure the snapshot status is `completed` before submitting the task.

Use below given AWS Credentials: (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

| Console URL | [https://284304506227.signin.aws.amazon.com/console?region=us-east-1](https://284304506227.signin.aws.amazon.com/console?region=us-east-1) |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Username    | kk\_labs\_user\_956360                                                                                                                     |
| Password    | WNb^I@ZtP^!1                                                                                                                               |
| Start Time  | Thu Feb 26 05:02:08 UTC 2026                                                                                                               |
| End Time    | Thu Feb 26 06:02:08 UTC 2026                                                                                                               |

\
`Notes:`

* Create the resources only in `us-east-1` region.
* To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:\
  ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)

<figure><img src=".gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>


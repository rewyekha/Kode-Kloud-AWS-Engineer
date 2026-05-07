# Day 12: Attach Volume to EC2 Instance

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.

An instance named `datacenter-ec2` and a volume named `datacenter-volume` already exists in `us-east-1` region. Attach the `datacenter-volume` volume to the `datacenter-ec2` instance, make sure to set the device name to `/dev/sdb` while attaching the volume.

Use below given AWS Credentials: (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

| Console URL | https://<aws-account>.signin.aws.amazon.com/console?region=us-east-1 |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Username    | <REDACTED_USERNAME> |
| Password    | <REDACTED_PASSWORD> |
| Start Time  | Mon Feb 23 08:48:30 UTC 2026                                                                                                               |
| End Time    | Mon Feb 23 09:48:30 UTC 2026                                                                                                               |

`Notes:`

* Create the resources only in `us-east-1` region.

<figure><img src=".gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

---

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

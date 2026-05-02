# Day 22: Configuring Secure SSH Access to an EC2 Instance

The Nautilus DevOps team needs to set up a new EC2 instance that can be accessed securely from their landing host (`aws-client`). The instance should be of type `t2.micro` and named `devops-ec2`. A new SSH key with name `id_rsa` should be created on the `aws-client` host under the`/root/.ssh/` folder, if it doesn't already exist. This key should then be added to the `root` user's authorised keys on the EC2 instance, allowing passwordless SSH access from the `aws-client` host.

**Notes:**

* Create the resources only in `us-east-1` region.
 ***

### Solution

#### Step 1: Generate SSH Key on aws-client Host

Check if SSH key exists:

```bash
~ on ☁️  (us-east-1) ➜  ls -l /root/.ssh/id_rsa
ls: cannot access '/root/.ssh/id_rsa': No such file or directory
```

Since it doesn't exist, generate a new SSH key pair:

```bash
~ on ☁️  (us-east-1) ✖ ssh-keygen -t rsa -b 2048 -f /root/.ssh/id_rsa -N ""
Generating public/private rsa key pair.
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:mdhwerEi8xjI4K9GvGwBZgbWdfk9PkVhbUFeDEmm2vk root@aws-client
The key's randomart image is:
+---[RSA 2048]----+
|  . .. ..    +B*o|
|.. .  ..    .+ooo|
|+     . + . o .. |
|o* .   * * = o   |
|*.o + + S o =    |
| +.  * o   o .   |
|o o.. .     . E  |
| =.              |
|o.               |
+----[SHA256]-----+
```

View the public key content (copy this for the next step):

```bash
~ on ☁️  (us-east-1) ➜  cat /root/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCXhGC/Iw4TxsA9vJpwV8jHg51/VDeUuzQtaNQU/TIpwR/hnswRR0vzv1puOWbSc0kTHuVTuDfOgkpM6EvbpTyvSxqMVEmn1OaR4RHnXRySYBzsnazL9p8wS++0OFNV6datuTnX/pPQxRvsybAgTf59HKvH3cPaM3IEc0PyftJoyCE606KieE06nN42XnwQzFL1JUD6tiR2PXf274yFgIYKEkaW5E5LdZ34oCakoZAAhKr5d5J8Df1tQzaYKf8iL6nrY+QYGeQ9t6r4rgvOmL5FfXiq0Yh27q9oXn1C3NKUQJzb4XfRSwpAAp6bTi8GNsXluf2w+8CBMJWSDjsiwgAD root@aws-client
```

***

#### Step 2: Launch EC2 Instance via AWS Console

* Log into the AWS Console at the provided URL with credentials.
* Set the region to **us-east-1**.
* Navigate to **EC2 → Instances → Launch Instances**.
* Configure as follows:
  * **Name**: `devops-ec2`
  * **AMI**: Amazon Linux 2 (default)
  * **Instance Type**: `t2.micro`
  * **Key pair**: Select **Proceed without a key pair** (we will add the key manually)
* Expand **Advanced Details** → paste the following **User Data** script (replace the public key part with your copied public key exactly):

```bash
#!/bin/bash
mkdir -p /root/.ssh
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCXhGC/Iw4TxsA9vJpwV8jHg51/VDeUuzQtaNQU/TIpwR/hnswRR0vzv1puOWbSc0kTHuVTuDfOgkpM6EvbpTyvSxqMVEmn1OaR4RHnXRySYBzsnazL9p8wS++0OFNV6datuTnX/pPQxRvsybAgTf59HKvH3cPaM3IEc0PyftJoyCE606KieE06nN42XnwQzFL1JUD6tiR2PXf274yFgIYKEkaW5E5LdZ34oCakoZAAhKr5d5J8Df1tQzaYKf8iL6nrY+QYGeQ9t6r4rgvOmL5FfXiq0Yh27q9oXn1C3NKUQJzb4XfRSwpAAp6bTi8GNsXluf2w+8CBMJWSDjsiwgAD root@aws-client" >> /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys
chown root:root /root/.ssh/authorized_keys
```

* Configure security group to allow **SSH (port 22)** from your landing host IP.
* Launch the instance.

***

#### Step 3: SSH into the Instance from aws-client

Use the new key to SSH into the instance (replace `<EC2_PUBLIC_IP>` with your instance’s public IP):

```bash
~ on ☁️  (us-east-1) ➜  ssh -i /root/.ssh/id_rsa root@<EC2_PUBLIC_IP>
The authenticity of host '3.80.77.120 (3.80.77.120)' can't be established.
ECDSA key fingerprint is SHA256:VRWhJeaV/iYvaSKpjjEh/npMD5cyP9cfDevSk3gBFCw.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '3.80.77.120' (ECDSA) to the list of known hosts.
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
[root@ip-172-31-27-13 ~]#
```

You are successfully logged in to the `devops-ec2` instance as root **without password prompt**.

***

### Summary

* Created SSH key on `aws-client`.
* Launched a `t2.micro` EC2 instance named `devops-ec2` with user data adding your public key.
* Allowed SSH inbound on security group for your IP.
* Connected securely via SSH with passwordless login.

Lab complete

<figure><img src=".gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>


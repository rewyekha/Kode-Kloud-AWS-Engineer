# Day 29: Establishing Secure Communication Between Public and Private VPCs via VPC Peering

***

## VPC Peering Between Public and Private VPCs

### Overview

This document demonstrates how to configure VPC Peering between a default public VPC and a private VPC to enable communication between EC2 instances in both networks.

***

### Architecture

* Public VPC (Default)
  * EC2 Instance: `devops-public-ec2`
* Private VPC
  * VPC Name: `devops-private-vpc`
  * CIDR: `10.1.0.0/16`
  * Subnet: `devops-private-subnet` (`10.1.1.0/24`)
  * EC2 Instance: `devops-private-ec2`
* VPC Peering Connection
  * Name: `devops-vpc-peering`

***

### Step 1: Create VPC Peering Connection

1. Navigate to **VPC Dashboard → Peering Connections**
2. Click **Create Peering Connection**

**Configuration:**

* Name: `devops-vpc-peering`
* Requester VPC: Default VPC
* Accepter VPC: `devops-private-vpc`

3. Create the connection
4. Select the peering connection → **Actions → Accept Request**

**Status:** Active

***

### Step 2: Update Route Tables

#### Public VPC Route Table

Add route:

* Destination: `10.1.0.0/16`
* Target: `devops-vpc-peering`

***

#### Private VPC Route Table

Add route:

* Destination: Default VPC CIDR (e.g., `172.31.0.0/16`)
* Target: `devops-vpc-peering`

***

### Step 3: Update Security Groups

#### Private EC2 Security Group

Add inbound rule:

* Type: All ICMP (IPv4)
* Source: Default VPC CIDR (e.g., `172.31.0.0/16`)

***

### Step 4: Retrieve Public Key from AWS Client

On the AWS client machine:

```bash
cat /root/.ssh/id_rsa.pub
```

#### Output

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC7examplekeycontent user@aws-client
```

***

### Step 5: Add Public Key to Public EC2

Connect to the public EC2 instance using EC2 Instance Connect or SSH.

Edit authorized keys:

```bash
vi ~/.ssh/authorized_keys
```

Paste the public key into the file.

Set correct permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

***

### Step 6: SSH into Public EC2 from AWS Client

```bash
ssh -i /root/.ssh/id_rsa ec2-user@3.91.219.13
```

#### First-time Connection Output

```bash
The authenticity of host '3.91.219.13 (3.91.219.13)' can't be established.
ECDSA key fingerprint is SHA256:Jc1E6kICOqBO2IkX7Skq8NNX+Hdujjh5dMPJVlngGkQ.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '3.91.219.13' (ECDSA) to the list of known hosts.
```

#### Successful Login

```bash
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

Last login: Sat Mar 21 23:57:11 2026 from 18.206.107.29
[ec2-user@ip-172-31-24-213 ~]$
```

***

### Step 7: Test Connectivity to Private EC2

Run ping from public EC2:

```bash
ping 10.1.1.242
```

#### Output

```bash
PING 10.1.1.242 (10.1.1.242) 56(84) bytes of data.
64 bytes from 10.1.1.242: icmp_seq=1 ttl=127 time=1.20 ms
64 bytes from 10.1.1.242: icmp_seq=2 ttl=127 time=0.788 ms
64 bytes from 10.1.1.242: icmp_seq=3 ttl=127 time=0.775 ms
64 bytes from 10.1.1.242: icmp_seq=4 ttl=127 time=0.818 ms
64 bytes from 10.1.1.242: icmp_seq=5 ttl=127 time=0.764 ms
64 bytes from 10.1.1.242: icmp_seq=6 ttl=127 time=0.915 ms
64 bytes from 10.1.1.242: icmp_seq=7 ttl=127 time=0.889 ms
64 bytes from 10.1.1.242: icmp_seq=8 ttl=127 time=0.744 ms
64 bytes from 10.1.1.242: icmp_seq=9 ttl=127 time=0.749 ms
64 bytes from 10.1.1.242: icmp_seq=10 ttl=127 time=0.814 ms
^C
--- 10.1.1.242 ping statistics ---
18 packets transmitted, 18 received, 0% packet loss, time 17539ms
rtt min/avg/max/mdev = 0.744/0.819/1.196/0.101 ms
```

***

### Verification Checklist

* VPC Peering status is Active
* Routes are configured in both VPCs
* Security group allows ICMP traffic
* SSH access to public EC2 is successful
* Ping from public EC2 to private EC2 is successful

***

### Conclusion

The VPC Peering connection has been successfully established between the default public VPC and the private VPC. Routing and security configurations allow seamless communication between EC2 instances across both VPCs.

This setup demonstrates secure private communication without requiring internet gateways or NAT configurations.

***

<figure><img src=".gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>

# Day 3: Create Subnet

The Nautilus DevOps team is migrating infrastructure to AWS. For this task, create one subnet named `datacenter-subnet` under the default VPC.

**Notes:**

* Create the resources only in `us-east-1` region.

---

## AWS CLI Steps

```bash
aws ec2 describe-vpcs \
  --filters Name=isDefault,Values=true \
  --region us-east-1 \
  --query "Vpcs[0].VpcId" \
  --output text
vpc-02fed80ae8f7dfe05

aws ec2 create-subnet \
  --vpc-id vpc-02fed80ae8f7dfe05 \
  --cidr-block 172.31.100.0/24 \
  --availability-zone us-east-1a \
  --region us-east-1
{
    "Subnet": {
        "AvailabilityZoneId": "use1-az6",
        "MapCustomerOwnedIpOnLaunch": false,
        "OwnerId": "074952969475",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "SubnetArn": "arn:aws:ec2:us-east-1:074952969475:subnet/subnet-09819702f88a771c7",
        "EnableDns64": false,
        "Ipv6Native": false,
        "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
        },
        "SubnetId": "subnet-09819702f88a771c7",
        "State": "available",
        "VpcId": "vpc-02fed80ae8f7dfe05",
        "CidrBlock": "172.31.100.0/24",
        "AvailableIpAddressCount": 251,
        "AvailabilityZone": "us-east-1a",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false
    }
}

aws ec2 create-tags \
  --resources subnet-09819702f88a771c7 \
  --tags Key=Name,Value=datacenter-subnet \
  --region us-east-1

aws ec2 describe-subnets \
  --filters Name=tag:Name,Values=datacenter-subnet \
  --region us-east-1 \
  --query "Subnets[*].[SubnetId,VpcId,CidrBlock]" \
  --output table
--------------------------------------------------------------------------
|                             DescribeSubnets                            |
+--------------------------+-------------------------+-------------------+
|  subnet-09819702f88a771c7|  vpc-02fed80ae8f7dfe05  |  172.31.100.0/24  |
+--------------------------+-------------------------+-------------------+
```

<figure><img src=".gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

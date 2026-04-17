# Day 43: Scaling and Managing Kubernetes Clusters with Amazon EKS

The Nautilus DevOps team has been tasked with preparing the infrastructure for a new Kubernetes-based application that will be deployed using Amazon EKS. The team is in the process of setting up an EKS cluster that meets their internal security and scalability standards. They require that the cluster be provisioned using the latest stable Kubernetes version to take advantage of new features and security improvements.

To minimize external exposure, the EKS cluster endpoint must be kept private. Additionally, the cluster needs to use the default VPC with availability zones `a`, `b`, and `c` to ensure high availability across different physical locations.

Your task is to create an EKS cluster named `xfusion-eks`, with Custom configuration, use IAM role for the cluster named `eksClusterRole`. Additionally, ensure that `EKS Auto Mode` is disabled and that the cluster endpoint access is set to private.

Finally, verify that the EKS cluster is successfully created with the correct configuration and is ready for workloads.\
`Notes:`

* Create the resources only in `us-east-1` region.
* To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:<br>



## Amazon EKS Cluster Provisioning (CLI Guide)

### Cluster: xfusion-eks (Private Endpoint, Default VPC)

***

### Overview

This document describes the step-by-step process to create an Amazon EKS cluster named **xfusion-eks** using AWS CLI. The cluster is configured with:

* Latest Kubernetes version (1.35)
* Default VPC
* Subnets across Availability Zones us-east-1a, us-east-1b, us-east-1c
* Private endpoint only (no public access)
* IAM role: eksClusterRole
* Region: us-east-1

***

### 1. Verify IAM Role (Initial Failure)

```bash
aws iam get-role --role-name eksClusterRole
```

Output:

```
An error occurred (NoSuchEntity) when calling the GetRole operation: The role with name eksClusterRole cannot be found.
```

***

### 2. Create IAM Role for EKS Cluster

#### Create trust policy

```bash
cat <<EOF > eks-trust-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
```

***

#### Create IAM role

```bash
aws iam create-role \
  --role-name eksClusterRole \
  --assume-role-policy-document file://eks-trust-policy.json
```

Output:

```
{
    "Role": {
        "Path": "/",
        "RoleName": "eksClusterRole",
        "RoleId": "AROA56W4NERYBPECTAYKK",
        "Arn": "arn:aws:iam::959313683568:role/eksClusterRole",
        "CreateDate": "2026-04-17T17:04:31Z",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "eks.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        }
    }
}
```

***

### 3. Attach Required Policy

```bash
aws iam attach-role-policy \
  --role-name eksClusterRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
```

(No output expected)

***

### 4. Verify IAM Role

```bash
aws iam get-role --role-name eksClusterRole
```

Output:

```
{
    "Role": {
        "Path": "/",
        "RoleName": "eksClusterRole",
        "RoleId": "AROA56W4NERYBPECTAYKK",
        "Arn": "arn:aws:iam::959313683568:role/eksClusterRole",
        "CreateDate": "2026-04-17T17:04:31Z",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "eks.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        },
        "MaxSessionDuration": 3600,
        "RoleLastUsed": {}
    }
}
```

***

### 5. Retrieve AWS Account ID

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
echo $ACCOUNT_ID
```

Output:

```
959313683568
```

***

### 6. Retrieve Latest Kubernetes Version

```bash
K8S_VERSION=$(aws eks describe-cluster-versions \
  --query "clusterVersions[].clusterVersion" \
  --output text | tr '\t' '\n' | sort -V | tail -1)

echo $K8S_VERSION
```

Output:

```
1.35
```

***

### 7. Get Default VPC

```bash
VPC_ID=$(aws ec2 describe-vpcs \
  --filters Name=isDefault,Values=true \
  --query "Vpcs[0].VpcId" \
  --output text)

echo $VPC_ID
```

Output:

```
vpc-071d855b6390f1df4
```

***

### 8. Get Subnets in AZ a, b, c

```bash
SUBNETS=$(aws ec2 describe-subnets \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query "Subnets[?AvailabilityZone=='us-east-1a' || AvailabilityZone=='us-east-1b' || AvailabilityZone=='us-east-1c'].SubnetId" \
  --output text)

echo $SUBNETS
```

Output:

```
subnet-07c78c23f7912e3df subnet-082c41b280ae504ea subnet-06c0739eaef36fdc0
```

***

### 9. Convert Subnets to CSV Format

```bash
SUBNETS_CSV=$(echo $SUBNETS | tr ' ' ',')
echo $SUBNETS_CSV
```

Output:

```
subnet-07c78c23f7912e3df,subnet-082c41b280ae504ea,subnet-06c0739eaef36fdc0
```

***

### 10. Create EKS Cluster

```bash
aws eks create-cluster \
  --name xfusion-eks \
  --region us-east-1 \
  --kubernetes-version $K8S_VERSION \
  --role-arn arn:aws:iam::$ACCOUNT_ID:role/eksClusterRole \
  --resources-vpc-config subnetIds=$SUBNETS_CSV,endpointPublicAccess=false,endpointPrivateAccess=true
```

Output:

```
{
    "cluster": {
        "name": "xfusion-eks",
        "arn": "arn:aws:eks:us-east-1:959313683568:cluster/xfusion-eks",
        "createdAt": 1776445726.347,
        "version": "1.35",
        "roleArn": "arn:aws:iam::959313683568:role/eksClusterRole",
        "resourcesVpcConfig": {
            "subnetIds": [
                "subnet-07c78c23f7912e3df",
                "subnet-082c41b280ae504ea",
                "subnet-06c0739eaef36fdc0"
            ],
            "vpcId": "vpc-071d855b6390f1df4",
            "endpointPublicAccess": false,
            "endpointPrivateAccess": true
        },
        "status": "CREATING"
    }
}
```

***

### 11. Wait for Cluster to Become Active

```bash
aws eks wait cluster-active --name xfusion-eks
```

(No output expected)

***

### 12. Verify Cluster Status

```bash
aws eks describe-cluster \
  --name xfusion-eks \
  --query "cluster.status"
```

Output:

```
"ACTIVE"
```

***

### 13. Verify Kubernetes Version

```bash
aws eks describe-cluster \
  --name xfusion-eks \
  --query "cluster.version"
```

Output:

```
"1.35"
```

***

### 14. Verify Endpoint Configuration

```bash
aws eks describe-cluster \
  --name xfusion-eks \
  --query "cluster.resourcesVpcConfig.{Private:endpointPrivateAccess,Public:endpointPublicAccess}"
```

Output:

```
{
    "Private": true,
    "Public": false
}
```

***

### 15. Final Validation Summary

| Parameter          | Value              |
| ------------------ | ------------------ |
| Cluster Name       | xfusion-eks        |
| Region             | us-east-1          |
| Kubernetes Version | 1.35               |
| VPC                | Default VPC        |
| Subnets            | us-east-1a, 1b, 1c |
| Endpoint Access    | Private only       |
| IAM Role           | eksClusterRole     |
| Status             | ACTIVE             |

***

### Conclusion

The Amazon EKS cluster **xfusion-eks** has been successfully provisioned using AWS CLI with secure private endpoint access and multi-AZ subnet configuration. The cluster is fully active and ready for workload deployment.

<figure><img src=".gitbook/assets/image (106).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (107).png" alt=""><figcaption></figcaption></figure>

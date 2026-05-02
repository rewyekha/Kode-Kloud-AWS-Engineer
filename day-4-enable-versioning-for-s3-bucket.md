# Day 4: Enable Versioning for S3 Bucket

Data protection and recovery are fundamental aspects of data management. It's essential to have systems in place to ensure that data can be recovered in case of accidental deletion or corruption. The DevOps team has received a requirement for implementing such measures for one of the S3 buckets they are managing.

The s3 bucket name is `nautilus-s3-16746`, enable `versioning` for this bucket.

> **Note:** Run `showcreds` on the `aws-client` host to retrieve temporary AWS credentials, then configure the AWS CLI using `aws configure`.

**Notes:**

* Create the resources only in `us-east-1` region.

<figure><img src=".gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

## Solution: Enable Versioning on S3 Bucket Using AWS CLI

### Step 1: Configure AWS CLI Credentials

On the `aws-client` host, configure AWS credentials:

```bash
aws configure
```

Enter the credentials obtained from the `showcreds` command:

```
AWS Access Key ID:     <from showcreds>
AWS Secret Access Key: <from showcreds>
Default region name:   us-east-1
Default output format: json
```

***

### Step 2: Verify the S3 Bucket Exists

Run the following command to ensure the bucket exists and is accessible:

```bash
aws s3api head-bucket --bucket nautilus-s3-16746
```

* No output indicates the bucket exists and access is permitted.

***

### Step 3: Enable Versioning on the S3 Bucket

Enable versioning using the AWS CLI:

```bash
aws s3api put-bucket-versioning \
  --bucket nautilus-s3-16746 \
  --versioning-configuration Status=Enabled \
  --region us-east-1
```

***

### Step 4: Verify Versioning Status

Confirm that versioning is enabled:

```bash
aws s3api get-bucket-versioning \
  --bucket nautilus-s3-16746 \
  --region us-east-1
```

Expected output:

```json
{
  "Status": "Enabled"
}
```

***

### Result

* Versioning has been **successfully enabled**
* Bucket: **nautilus-s3-16746**
* Region: **us-east-1**

***

### Notes

* S3 versioning is a **bucket-level setting** and applies to all objects
* Once enabled, versioning **cannot be disabled**, only suspended
* Existing objects will remain unversioned until modified

```
~ on ☁️  (us-east-1) ➜  aws s3api head-bucket --bucket nautilus-s3-16746
{
    "BucketRegion": "us-east-1",
    "AccessPointAlias": false
}

~ on ☁️  (us-east-1) ➜  aws s3api get-bucket-versioning \
  --bucket nautilus-s3-16746 \
  --region us-east-1
{
    "Status": "Enabled"
}

~ on ☁️  (us-east-1) ➜
```

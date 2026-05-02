# Day 39: Hosting a Static Website on AWS S3

The Nautilus DevOps team has been tasked with creating an internal information portal for public access. As part of this project, they need to host a static website on AWS using an S3 bucket. The S3 bucket must be configured for public access to allow external users to access the static website directly via the S3 website URL.

Task Requirements:

1. Create an S3 bucket named `xfusion-web-30228`.
2. Configure the S3 bucket for static website hosting with `index.html` as the index document.
3. Allow public access to the bucket so that the website is publicly accessible.
4. Upload the `index.html` file from the `/root/` directory of the AWS client host to the S3 bucket.
5. Verify that the website is accessible directly through the S3 website URL.

\
`Notes:`

* Create the resources only in `us-east-1` region.
* To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:\
  ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)





## AWS S3: Host a Static Website with Public Access

> **Platform:** KodeKloud | **Cloud:** AWS | **Region:** us-east-1 **Difficulty:** Beginner | **Topic:** AWS S3, Static Website Hosting, Bucket Policy, Public Access

***

### Table of Contents

1. [Lab Overview](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-overview)
2. [Lab Objectives](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-objectives)
3. [Prerequisites](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#prerequisites)
4. [Architecture](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#architecture)
5. [Solution](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#solution)
   * [Step 1: Verify the Source File](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-verify-the-source-file)
   * [Step 2: Create the S3 Bucket](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-create-the-s3-bucket)
   * [Step 3: Disable Public Access Block](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-disable-public-access-block)
   * [Step 4: Create and Apply the Bucket Policy](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-create-and-apply-the-bucket-policy)
   * [Step 5: Enable Static Website Hosting](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-enable-static-website-hosting)
   * [Step 6: Upload index.html to the Bucket](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-upload-indexhtml-to-the-bucket)
   * [Step 7: Verify the Upload](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-7-verify-the-upload)
   * [Step 8: Verify Website is Publicly Accessible](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-8-verify-website-is-publicly-accessible)
6. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
7. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)
8. [Resource Reference](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#resource-reference)

***

### Lab Overview

The Nautilus DevOps team has been tasked with creating an internal information portal for public access. As part of this project, they need to host a static website on AWS using an S3 bucket. The S3 bucket must be configured for public access to allow external users to access the static website directly via the S3 website URL.

***

### Lab Objectives

1. Create an S3 bucket named `xfusion-web-30228`.
2. Configure the S3 bucket for static website hosting with `index.html` as the index document.
3. Allow public access to the bucket so the website is publicly accessible.
4. Upload the `index.html` file from `/root/` on the AWS client host to the S3 bucket.
5. Verify the website is accessible through the S3 website URL.

***

### Prerequisites

| Field         | Value                                                                 |
| ------------- | --------------------------------------------------------------------- |
| Console URL   | `https://230266393235.signin.aws.amazon.com/console?region=us-east-1` |
| Username      | `kk_labs_user_915151`                                                 |
| Password      | `URD2k0C%1RId`                                                        |
| Region        | `us-east-1`                                                           |
| Access Method | AWS CLI on `aws-client` host                                          |
| Source file   | `/root/index.html`                                                    |
| Bucket name   | `xfusion-web-30228`                                                   |

***

### Architecture

```bash
aws-client host
      │
      │ aws s3 cp /root/index.html
      │
      ▼
┌─────────────────────────────────────────────────────┐
│          S3 Bucket: xfusion-web-30228               │
│          Region: us-east-1                          │
│                                                     │
│  Static Website Hosting: Enabled                    │
│  Index Document: index.html                         │
│  Public Access Block: Disabled                      │
│  Bucket Policy: s3:GetObject → Principal: *         │
│                                                     │
│  Objects:                                           │
│    index.html (20 bytes)                            │
└──────────────────────┬──────────────────────────────┘
                       │
                       │ HTTP 200 OK
                       ▼
         http://xfusion-web-30228.s3-website-us-east-1.amazonaws.com
                       │
                       ▼
              Browser / curl client
```

***

### Solution

#### Step 1: Verify the Source File

Confirm that `index.html` exists in the `/root/` directory on the aws-client host before creating any resources:

```bash
cd /root
ls
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  cd /root
~ on ☁️  (us-east-1) ➜  ls
index.html
```

The file `index.html` is present and ready for upload.

***

#### Step 2: Create the S3 Bucket

Create the S3 bucket in `us-east-1`. Note that `us-east-1` does not require a `--create-bucket-configuration` flag — it is the default region for S3 bucket creation:

```bash
aws s3api create-bucket \
  --bucket xfusion-web-30228 \
  --region us-east-1
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  aws s3api create-bucket \
  --bucket xfusion-web-30228 \
  --region us-east-1
{
    "Location": "/xfusion-web-30228"
}
```

The bucket `xfusion-web-30228` was created successfully. The `Location` field confirms the bucket name and that it was created in the default region.

***

#### Step 3: Disable Public Access Block

By default, AWS S3 buckets have all four public access block settings enabled — which would prevent the bucket policy and the website from being publicly accessible. All four settings must be explicitly disabled:

```bash
aws s3api put-public-access-block \
  --bucket xfusion-web-30228 \
  --public-access-block-configuration \
    BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  aws s3api put-public-access-block \
  --bucket xfusion-web-30228 \
  --public-access-block-configuration \
    BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false

~ on ☁️  (us-east-1) ➜
```

No output means the command succeeded. All four public access block settings are now disabled, allowing the bucket policy to grant public read access.

***

#### Step 4: Create and Apply the Bucket Policy

Create a bucket policy that grants `s3:GetObject` permission to all principals (`"Principal": "*"`) — making every object in the bucket publicly readable:

```bash
cat > policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::xfusion-web-30228/*"
    }
  ]
}
EOF
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  cat > policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::xfusion-web-30228/*"
    }
  ]
}
EOF

~ on ☁️  (us-east-1) ➜
```

Apply the policy to the bucket:

```bash
aws s3api put-bucket-policy \
  --bucket xfusion-web-30228 \
  --policy file://policy.json
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  aws s3api put-bucket-policy \
  --bucket xfusion-web-30228 \
  --policy file://policy.json

~ on ☁️  (us-east-1) ➜
```

No output means the policy was applied successfully. The bucket now allows anonymous read access to all objects.

***

#### Step 5: Enable Static Website Hosting

Configure the bucket as a static website host and set `index.html` as the index document:

```bash
aws s3 website s3://xfusion-web-30228/ \
  --index-document index.html
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  aws s3 website s3://xfusion-web-30228/ \
  --index-document index.html

~ on ☁️  (us-east-1) ➜
```

No output means the website hosting configuration was applied successfully. The bucket now has a website endpoint at:

```
http://xfusion-web-30228.s3-website-us-east-1.amazonaws.com
```

***

#### Step 6: Upload index.html to the Bucket

Upload the `index.html` file from the aws-client host to the S3 bucket.

The first attempt used `--acl public-read` which is not supported when ACLs are disabled on the bucket (which is the default for newly created buckets):

```bash
aws s3 cp /root/index.html s3://xfusion-web-30228/ --acl public-read
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ✖ aws s3 cp /root/index.html s3://xfusion-web-30228/ \
  --acl public-read
upload failed: ./index.html to s3://xfusion-web-30228/index.html An error occurred (AccessControlListNotSupported) when calling the PutObject operation: The bucket does not allow ACLs
```

The `--acl public-read` flag is not needed because the bucket policy already grants public read access to all objects. Upload without the ACL flag:

```bash
aws s3 cp /root/index.html s3://xfusion-web-30228/
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  aws s3 cp /root/index.html s3://xfusion-web-30228/
upload: ./index.html to s3://xfusion-web-30228/index.html
```

The file uploaded successfully. The bucket policy handles public access — no per-object ACL is needed.

***

#### Step 7: Verify the Upload

Confirm the file was uploaded correctly and note the file size:

```bash
aws s3 ls s3://xfusion-web-30228/
```

**Terminal Output:**

```
~ on ☁️  (us-east-1) ➜  aws s3 ls s3://xfusion-web-30228/
2026-04-10 04:01:39         20 index.html
```

The file `index.html` is present in the bucket — `20 bytes`, uploaded at `04:01:39 UTC`.

***

#### Step 8: Verify Website is Publicly Accessible

Send an HTTP request to the S3 website endpoint to confirm it returns `HTTP 200 OK`:

```bash
curl -I http://xfusion-web-30228.s3-website-us-east-1.amazonaws.com
```

**Terminal Output:**

```bash
~ on ☁️  (us-east-1) ➜  curl -I http://xfusion-web-30228.s3-website-us-east-1.amazonaws.com
HTTP/1.1 200 OK
x-amz-id-2: 85FsxCOwMUQkl5wIUMgJTw9QR/XEJs74flFp3V2B+hiUVeux881Dwt8kvK0H2d868uDw1BcNjMLDZge3TD8ypbBMMJzNbwAs
x-amz-request-id: H46NR0NB28W9CJQ5
Date: Fri, 10 Apr 2026 04:01:54 GMT
Last-Modified: Fri, 10 Apr 2026 04:01:39 GMT
ETag: "bddd508bfd1fab1d7eabc0ad6d8db1ea"
Content-Type: text/html
Content-Length: 20
Server: AmazonS3

~ on ☁️  (us-east-1) ➜
```

The S3 static website returned `HTTP/1.1 200 OK` confirming:

* The bucket is configured correctly for static website hosting
* The `index.html` file is publicly accessible
* Content-Type is `text/html` and Content-Length is `20` bytes — matching the uploaded file

***

### Lab Complete

| Requirement                    | Detail                                    | Status    |
| ------------------------------ | ----------------------------------------- | --------- |
| Bucket created                 | `xfusion-web-30228` in `us-east-1`        | Confirmed |
| Public access block disabled   | All four settings set to `false`          | Confirmed |
| Bucket policy applied          | `s3:GetObject` for `Principal: *`         | Confirmed |
| Static website hosting enabled | Index document: `index.html`              | Confirmed |
| File uploaded                  | `index.html` — 20 bytes at `04:01:39 UTC` | Confirmed |
| Website accessible             | `HTTP/1.1 200 OK` from S3 website URL     | Confirmed |

***

### Key Concepts

#### Why ACL Upload Failed — Bucket Policy vs Object ACL

The first upload attempt failed with `AccessControlListNotSupported`:

```
--acl public-read → FAILED: The bucket does not allow ACLs
```

Modern S3 buckets enforce **Object Ownership** set to `BucketOwnerEnforced` by default, which disables ACLs entirely. Two methods exist for granting public read access:

| Method                                                | When to Use                                            |
| ----------------------------------------------------- | ------------------------------------------------------ |
| **Object ACL** (`--acl public-read`)                  | Only when ACLs are enabled on the bucket (legacy mode) |
| **Bucket Policy** (`s3:GetObject` for `Principal: *`) | Recommended — works regardless of ACL settings         |

In this lab the bucket policy was the correct approach. Once the policy grants `s3:GetObject` to `Principal: *`, all objects in the bucket are publicly readable without any per-object ACL.

#### The Four Public Access Block Settings

AWS introduced four independent settings to control public access at the account and bucket level:

| Setting                 | What it Blocks                                    |
| ----------------------- | ------------------------------------------------- |
| `BlockPublicAcls`       | Prevents adding public ACLs to objects            |
| `IgnorePublicAcls`      | Ignores existing public ACLs on objects           |
| `BlockPublicPolicy`     | Prevents bucket policies that grant public access |
| `RestrictPublicBuckets` | Blocks public access even if policy exists        |

All four must be set to `false` for a bucket policy granting public access to take effect. Leaving any one enabled can silently prevent the policy from working.

#### S3 Website Endpoint vs S3 REST Endpoint

S3 provides two distinct URL formats:

| Endpoint Type    | Format                                                        | Returns                                                   |
| ---------------- | ------------------------------------------------------------- | --------------------------------------------------------- |
| REST endpoint    | `https://xfusion-web-30228.s3.amazonaws.com/index.html`       | Direct object access — requires auth for private objects  |
| Website endpoint | `http://xfusion-web-30228.s3-website-us-east-1.amazonaws.com` | Serves `index.html` for root requests, supports redirects |

The website endpoint is required for static website hosting because it understands the `index.html` concept — a GET to `/` returns the index document. The REST endpoint does not do this by default.

#### S3 Website URL Format

The website endpoint URL follows this pattern:

```bash
http://<bucket-name>.s3-website-<region>.amazonaws.com

Example:
http://xfusion-web-30228.s3-website-us-east-1.amazonaws.com
```

Note that S3 static website endpoints only support **HTTP** — not HTTPS. To serve over HTTPS, the website must be fronted by CloudFront with an SSL certificate.

#### Correct Order of Operations

The steps must be performed in the correct order to avoid permission errors:

```bash
1. Create bucket
2. Disable public access block  ← must happen before step 3
3. Apply bucket policy          ← grants public read
4. Enable website hosting
5. Upload file                  ← no --acl flag needed
6. Verify
```

If the bucket policy is applied before disabling the public access block, the `put-bucket-policy` call will fail with `AccessDenied` because `BlockPublicPolicy` is still enabled.

***

### Resource Reference

| Resource           | Type         | Value                                                         |
| ------------------ | ------------ | ------------------------------------------------------------- |
| Bucket name        | S3           | `xfusion-web-30228`                                           |
| Region             | AWS          | `us-east-1`                                                   |
| Index document     | S3 website   | `index.html`                                                  |
| Uploaded file size | Object       | `20 bytes`                                                    |
| Upload timestamp   | UTC          | `2026-04-10 04:01:39`                                         |
| File ETag          | MD5          | `bddd508bfd1fab1d7eabc0ad6d8db1ea`                            |
| Website URL        | S3           | `http://xfusion-web-30228.s3-website-us-east-1.amazonaws.com` |
| HTTP response      | Verification | `200 OK`                                                      |
| Request ID         | AWS          | `H46NR0NB28W9CJQ5`                                            |

***

_Lab completed on 2026-04-10 | AWS Region: us-east-1 | Platform: KodeKloud_

<figure><img src=".gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>


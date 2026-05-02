# Day 23: Data Migration Between S3 Buckets Using AWS CLI

As part of a data migration project, the team lead has tasked the team with migrating data from an existing S3 bucket to a new S3 bucket. The existing bucket contains a substantial amount of data that must be accurately transferred to the new bucket. The team is responsible for creating the new S3 bucket and ensuring that all data from the existing bucket is copied or synced to the new bucket completely and accurately. It is imperative to perform thorough verification steps to confirm that all data has been successfully transferred to the new bucket without any loss or corruption.

As a member of the Nautilus DevOps Team, your task is to perform the following:

Create a New Private S3 Bucket: Name the bucket `nautilus-sync-7916`.

Data Migration: Migrate the entire data from the existing `nautilus-s3-5348` bucket to the new `nautilus-sync-7916` bucket.

Ensure Data Consistency: Ensure that both buckets have the same data.

Use AWS CLI: Use the AWS CLI to perform the creation and data migration tasks.

**Notes:**

* Create the resources only in `us-east-1` region.

As part of a data migration project, the team lead has tasked the team with migrating data from an existing S3 bucket to a new S3 bucket.

The existing bucket contains a substantial amount of data that must be accurately transferred to the new bucket. The team is responsible for creating the new S3 bucket and ensuring that all data from the existing bucket is copied or synced completely and accurately.

It is imperative to perform thorough verification steps to confirm that all data has been successfully transferred without any loss or corruption.

#### Task Requirements

1. Create a new **private S3 bucket** named **`devops-sync-27893`**.
2. Migrate all data from the existing bucket **`devops-s3-13316`**.
3. Ensure both buckets contain the **same data**.
4. Use **AWS CLI** to perform the operations.
5. All resources must be created in **us-east-1 region**.

***

## Step 1: Verify Existing Buckets

First, list all available buckets to confirm the source bucket exists.

#### Command

```bash
aws s3 ls
```

#### Terminal Output

```bash
2025-03-13 09:12:18 devops-s3-13316
```

***

## Step 2: Create the New S3 Bucket

Create a new bucket named **devops-sync-27893** in the **us-east-1** region.

#### Command

```bash
aws s3api create-bucket \
--bucket devops-sync-27893 \
--region us-east-1
```

#### Terminal Output

```bash
{
    "Location": "/devops-sync-27893"
}
```

***

## Step 3: Verify the Bucket Creation

Confirm that the new bucket has been created successfully.

#### Command

```bash
aws s3 ls
```

#### Terminal Output

```bash
2025-03-13 09:12:18 devops-s3-13316
2025-03-13 09:14:02 devops-sync-27893
```

***

## Step 4: Migrate Data Between Buckets

Use the **AWS CLI sync command** to copy all data from the source bucket to the destination bucket.

#### Command

```bash
aws s3 sync s3://devops-s3-13316 s3://devops-sync-27893
```

#### Terminal Output

```bash
copy: s3://devops-s3-13316/app/config.json to s3://devops-sync-27893/app/config.json
copy: s3://devops-s3-13316/logs/log1.txt to s3://devops-sync-27893/logs/log1.txt
copy: s3://devops-s3-13316/images/img1.png to s3://devops-sync-27893/images/img1.png
copy: s3://devops-s3-13316/data/datafile.csv to s3://devops-sync-27893/data/datafile.csv
```

This command ensures:

* All objects are copied
* Folder structure is preserved
* Only missing or changed files are transferred

***

## Step 5: Verify Data in Source Bucket

Check the number of objects in the **source bucket**.

#### Command

```bash
aws s3 ls s3://devops-s3-13316 --recursive | wc -l
```

#### Terminal Output

```bash
4
```

***

## Step 6: Verify Data in Destination Bucket

Check the number of objects in the **new bucket**.

#### Command

```bash
aws s3 ls s3://devops-sync-27893 --recursive | wc -l
```

#### Terminal Output

```bash
4
```

The object counts match, indicating that the data migration was successful.

***

## Step 7: Final Validation Using Dry Run

Run the sync command again with **--dryrun** to confirm that no additional files need to be copied.

#### Command

```bash
aws s3 sync s3://devops-s3-13316 s3://devops-sync-27893 --dryrun
```

#### Terminal Output

```bash
(dryrun) No files to sync
```

This confirms both buckets contain identical data.

***

## Conclusion

The migration process was successfully completed using the **AWS CLI**.

#### Summary

* Created new S3 bucket **devops-sync-27893**
* Migrated all data from **devops-s3-13316**
* Verified object counts in both buckets
* Performed dry-run validation to confirm synchronization

Both buckets now contain **identical data**, ensuring a successful migration.

<figure><img src=".gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

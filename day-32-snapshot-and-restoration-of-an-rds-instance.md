# Day 32: Snapshot and Restoration of an RDS Instance

The Nautilus Development Team is preparing for a major update to their database infrastructure. To ensure a smooth transition and to safeguard data, the team has requested the DevOps team to take a snapshot of the current RDS instance and restore it to a new instance. This process is crucial for testing and validation purposes before the update is rolled out to the production environment. The snapshot will serve as a backup, and the new instance will be used to verify that the backup process works correctly and that the application can function seamlessly with the restored data.

As a member of the Nautilus DevOps Team, your task is to perform the following:

1. Take a Snapshot: Take a snapshot of the `devops-rds` RDS instance and name it `devops-snapshot` (please wait `devops-rds` instance to be in `available` state).
2. Restore the Snapshot: Restore the snapshot to a new RDS instance named `devops-snapshot-restore`.
3. Configure the New RDS Instance: Ensure that the new RDS instance has a class of `db.t3.micro`.
4. Verify the New RDS Instance: The new RDS instance must be in the `Available` state upon completion of the restoration process.

**Notes:**

* Create the resources only in `us-east-1` region.
* To `display` or `hide` the terminal of the AWS client machine, you can use the expand toggle button as shown below:\
  ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)
 ***

## RDS: Snapshot and Restore an Instance Using AWS CLI

> **Platform:** AWS | **Series:** Nautilus DevOps — Stratos Datacenter
> **Difficulty:** Intermediate | **Topic:** AWS RDS, Snapshots, Restore, AWS CLI

***

### Table of Contents

1. [Lab Question](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#lab-question)
2. [Infrastructure Details](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#infrastructure-details)
3. [Solution](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#solution)
   * [Step 1: Set Variables](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-1-set-variables)
   * [Step 2: Wait for Source RDS Instance](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-2-wait-for-source-rds-instance)
   * [Step 3: Create Snapshot](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-3-create-snapshot)
   * [Step 4: Wait for Snapshot Completion](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-4-wait-for-snapshot-completion)
   * [Step 5: Verify Snapshot Dependency Error](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-5-verify-snapshot-dependency-error)
   * [Step 6: Restore Snapshot to New Instance](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-6-restore-snapshot-to-new-instance)
   * [Step 7: Wait for Restored Instance](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-7-wait-for-restored-instance)
   * [Step 8: Verify Restored Instance](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-8-verify-restored-instance)
4. [Lab Complete](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#lab-complete)
5. [Key Concepts](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#key-concepts)

***

### Lab Question

The Nautilus Development Team is preparing for a major update to their database infrastructure. To ensure a smooth transition and to safeguard data, the team has requested the DevOps team to take a snapshot of the current RDS instance and restore it to a new instance.

Requirements:

1. Take a snapshot of `devops-rds` named `devops-snapshot`.
2. Restore the snapshot to a new instance named `devops-snapshot-restore`.
3. Ensure the new instance uses class `db.t3.micro`.
4. Verify the new instance is in `available` state.

***

### Infrastructure Details

| Component      | Value                   |
| -------------- | ----------------------- |
| Region         | us-east-1               |
| Source RDS     | devops-rds              |
| Snapshot Name  | devops-snapshot         |
| Target RDS     | devops-snapshot-restore |
| Instance Class | db.t3.micro             |

> **Target:** AWS RDS (us-east-1 region)

***

### Solution

#### Step 1: Set Variables

Define reusable variables for consistency and clarity.

```bash
SOURCE_RDS="devops-rds"
SNAPSHOT="devops-snapshot"
TARGET_RDS="devops-snapshot-restore"
CLASS="db.t3.micro"
REGION="us-east-1"
```

Terminal Output:

```bash
~ on ☁️  (us-east-1) ➜  SOURCE_RDS="devops-rds"
SNAPSHOT="devops-snapshot"
TARGET_RDS="devops-snapshot-restore"
CLASS="db.t3.micro"
REGION="us-east-1"
```

Variables are successfully initialized.

***

#### Step 2: Wait for Source RDS Instance

Ensure the source database is in available state before snapshot creation.

```bash
aws rds wait db-instance-available \
  --db-instance-identifier $SOURCE_RDS \
  --region $REGION
```

Terminal Output:

```bash
(no output)
```

No output indicates the instance is already available.

***

#### Step 3: Create Snapshot

Create a manual snapshot of the source RDS instance.

```bash
aws rds create-db-snapshot \
  --db-instance-identifier $SOURCE_RDS \
  --db-snapshot-identifier $SNAPSHOT \
  --region $REGION
```

Terminal Output:

```bash
{
    "DBSnapshot": {
        "DBSnapshotIdentifier": "devops-snapshot",
        "DBInstanceIdentifier": "devops-rds",
        "Engine": "mysql",
        "AllocatedStorage": 5,
        "Status": "creating",
        ...
        "DBSnapshotArn": "arn:aws:rds:us-east-1:438137422704:snapshot:devops-snapshot"
    }
}
```

Snapshot creation has started successfully.

***

#### Step 4: Wait for Snapshot Completion

Wait until snapshot becomes available.

```bash
aws rds wait db-snapshot-available \
  --db-snapshot-identifier $SNAPSHOT \
  --region $REGION
```

Terminal Output:

```bash
(no output)
```

Snapshot is now ready for restoration.

***

#### Step 5: Verify Snapshot Dependency Error

Attempting to check the restored instance before creation results in an error.

```bash
aws rds describe-db-instances \
  --db-instance-identifier $TARGET_RDS \
  --region $REGION \
  --query "DBInstances[0].[DBInstanceStatus,DBInstanceClass]" \
  --output table
```

Terminal Output:

```bash
An error occurred (DBInstanceNotFound) when calling the DescribeDBInstances operation: DBInstance devops-snapshot-restore not found.
```

This confirms the restore step has not yet been executed.

***

#### Step 6: Restore Snapshot to New Instance

Restore the snapshot to create a new RDS instance.

```bash
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier $TARGET_RDS \
  --db-snapshot-identifier $SNAPSHOT \
  --db-instance-class $CLASS \
  --region $REGION \
  --no-publicly-accessible
```

Terminal Output:

```bash
{
    "DBInstance": {
        "DBInstanceIdentifier": "devops-snapshot-restore",
        "DBInstanceClass": "db.t3.micro",
        "Engine": "mysql",
        "DBInstanceStatus": "creating",
        ...
        "PubliclyAccessible": false
    }
}
```

Restoration process has started successfully.

***

#### Step 7: Wait for Restored Instance

Wait until the new instance becomes available.

```bash
aws rds wait db-instance-available \
  --db-instance-identifier $TARGET_RDS \
  --region $REGION
```

Terminal Output:

```bash
(no output)
```

The restored instance is now available.

***

#### Step 8: Verify Restored Instance

Confirm instance status and configuration.

```bash
aws rds describe-db-instances \
  --db-instance-identifier $TARGET_RDS \
  --region $REGION \
  --query "DBInstances[0].[DBInstanceStatus,DBInstanceClass]" \
  --output table
```

Terminal Output:

```bash
-----------------------------------------
|  DescribeDBInstances                  |
+-------------+-------------------------+
| available   | db.t3.micro             |
+-------------+-------------------------+
```

The restored instance is successfully available with correct configuration.

***

### Lab Complete

| Task              | Detail                  | Status    |
| ----------------- | ----------------------- | --------- |
| Source instance   | devops-rds              | Available |
| Snapshot created  | devops-snapshot         | Completed |
| Snapshot status   | Available               | Verified  |
| Restore operation | devops-snapshot-restore | Completed |
| Instance class    | db.t3.micro             | Verified  |
| Final state       | available               | Confirmed |

***

### Key Concepts

#### RDS Snapshot

An RDS snapshot is a point-in-time backup of a database instance. It captures:

* Data
* Configuration
* Storage settings

Snapshots are used for backup and cloning purposes.

***

#### Snapshot vs Restore

| Operation | Purpose                           |
| --------- | --------------------------------- |
| Snapshot  | Backup existing database          |
| Restore   | Create new database from snapshot |

Snapshots do not create instances automatically; restore must be explicitly executed.

***

#### AWS CLI Wait Commands

AWS CLI provides waiters to block execution until a resource reaches a desired state.

```bash
aws rds wait db-instance-available
```

This ensures automation scripts run reliably without race conditions.

***

#### Instance Class

```bash
db.t3.micro
```

Defines CPU, memory, and performance characteristics of the database instance.

***

#### Restore Behavior

When restoring:

* VPC and subnet settings are inherited
* Security groups are reused
* Storage configuration remains consistent

Only specified parameters like instance class are overridden.

***

_Lab completed on 2026-03-25 | Region: us-east-1 | Service: AWS RDS | Tool: AWS CLI_

<figure><img src=".gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

# Day 42: Building and Managing NoSQL Databases with AWS DynamoDB

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

The Nautilus DevOps team is developing a simple 'To-Do' application using DynamoDB to store and manage tasks efficiently. The team needs to create a DynamoDB table to hold tasks, each identified by a unique task ID. Each task will have a description and a status, which indicates the progress of the task (e.g., 'completed' or 'in-progress').

Your task is to:

1. Create a DynamoDB table named `nautilus-tasks` with a primary key called `taskId` (string).
2. Insert the following tasks into the table:
   * Task 1: `taskId`: '1', description: 'Learn DynamoDB', status: 'completed'
   * Task 2: `taskId`: '2', description: 'Build To-Do App', status: 'in-progress'
3. Verify that Task 1 has a status of 'completed' and Task 2 has a status of 'in-progress'.

Ensure the DynamoDB table is created successfully and that both tasks are inserted correctly with the appropriate statuses.\
`Notes:`

* Create the resources only in `us-east-1` region.

---

## Nautilus To-Do App - DynamoDB Setup

**Lab Objective**\
Create a DynamoDB table `nautilus-tasks` and insert sample tasks for the Nautilus To-Do application using AWS CLI.

**Region**: `us-east-1`\
**Billing Mode**: Pay-per-request

---

### 1. Create the DynamoDB Table

**Command:**

```bash
aws dynamodb create-table \
  --table-name nautilus-tasks \
  --attribute-definitions AttributeName=taskId,AttributeType=S \
  --key-schema AttributeName=taskId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region us-east-1
```

**Output:**

```json
{
    "TableDescription": {
        "TableName": "nautilus-tasks",
        "TableStatus": "CREATING",
        ...
        "BillingModeSummary": {
            "BillingMode": "PAY_PER_REQUEST"
        }
    }
}
```

---

### 2. Verify Table Status

**Command:**

```bash
aws dynamodb describe-table \
  --table-name nautilus-tasks \
  --region us-east-1 \
  --query "Table.TableStatus"
```

**Expected Output:**

```
"ACTIVE"
```

---

### 3. Insert Tasks into the Table

#### Task 1: Learn DynamoDB

```bash
aws dynamodb put-item \
  --table-name nautilus-tasks \
  --item '{
    "taskId": {"S": "1"},
    "description": {"S": "Learn DynamoDB"},
    "status": {"S": "completed"}
  }' \
  --region us-east-1
```

#### Task 2: Build To-Do App

```bash
aws dynamodb put-item \
  --table-name nautilus-tasks \
  --item '{
    "taskId": {"S": "2"},
    "description": {"S": "Build To-Do App"},
    "status": {"S": "in-progress"}
  }' \
  --region us-east-1
```

---

### 4. Verify Inserted Tasks

#### Get Task 1

```bash
aws dynamodb get-item \
  --table-name nautilus-tasks \
  --key '{"taskId": {"S": "1"}}' \
  --region us-east-1
```

**Output:**

```json
{
    "Item": {
        "taskId": { "S": "1" },
        "description": { "S": "Learn DynamoDB" },
        "status": { "S": "completed" }
    }
}
```

#### Get Task 2

```bash
aws dynamodb get-item \
  --table-name nautilus-tasks \
  --key '{"taskId": {"S": "2"}}' \
  --region us-east-1
```

**Output:**

```json
{
    "Item": {
        "taskId": { "S": "2" },
        "description": { "S": "Build To-Do App" },
        "status": { "S": "in-progress" }
    }
}
```

---

### Summary of All Commands Used

```bash
# 1. Create Table
aws dynamodb create-table \
  --table-name nautilus-tasks \
  --attribute-definitions AttributeName=taskId,AttributeType=S \
  --key-schema AttributeName=taskId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region us-east-1

# 2. Check Table Status
aws dynamodb describe-table --table-name nautilus-tasks --region us-east-1 --query "Table.TableStatus"

# 3. Insert Task 1
aws dynamodb put-item \
  --table-name nautilus-tasks \
  --item '{
    "taskId": {"S": "1"},
    "description": {"S": "Learn DynamoDB"},
    "status": {"S": "completed"}
  }' \
  --region us-east-1

# 4. Insert Task 2
aws dynamodb put-item \
  --table-name nautilus-tasks \
  --item '{
    "taskId": {"S": "2"},
    "description": {"S": "Build To-Do App"},
    "status": {"S": "in-progress"}
  }' \
  --region us-east-1

# 5. Verify Task 1
aws dynamodb get-item \
  --table-name nautilus-tasks \
  --key '{"taskId": {"S": "1"}}' \
  --region us-east-1

# 6. Verify Task 2
aws dynamodb get-item \
  --table-name nautilus-tasks \
  --key '{"taskId": {"S": "2"}}' \
  --region us-east-1
```

---

### Lab Notes

* All operations were performed in the **`us-east-1`** region.
* Table uses **String** type for `taskId` as the partition key.
* Billing mode is set to **PAY\_PER\_REQUEST** (serverless).
* Even though the AWS Console may show permission errors (due to lab restrictions), the resources were successfully created and verified via CLI.

---

<figure><img src=".gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (102).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (104).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>

---

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

# Day 41: Securing Data with AWS KMS

The Nautilus DevOps team is focusing on improving their data security by using AWS KMS. Your task is to create a KMS key and manage the encryption and decryption of a pre-existing sensitive file using the KMS key.

Specific Requirements:

1. Create a symmetric KMS key named `xfusion-KMS-Key` to manage encryption and decryption.
2. Encrypt the provided `SensitiveData.txt` file (located in /root/), base64 encode the ciphertext, and save the encrypted version as `EncryptedData.bin` in the `/root/` directory.
3. Try to decrypt the same and verify that the decrypted data matches the original file.

Make sure that the KMS key is correctly configured. The validation script will test your configuration by decrypting the `EncryptedData.bin` file using the KMS key you created.
**Notes:**

* Create the resources only in `us-east-1` region.

## AWS KMS Encryption and Decryption Workflow

### Overview

This document describes the process of creating an AWS Key Management Service (KMS) key and using it to encrypt and decrypt a sensitive file. The workflow ensures secure handling of data using a symmetric encryption key.

The implementation uses the AWS CLI to:

* Create a KMS key
* Assign an alias
* Encrypt a file
* Decrypt the file
* Verify data integrity

***

### Prerequisites

* AWS CLI installed and configured
* Proper IAM permissions for KMS operations
*   File available at:

    ```
    /root/SensitiveData.txt
    ```
* AWS region configured (example: us-east-1)

***

### Step 1: Create KMS Key

Run the following command to create a symmetric encryption key:

```bash
KEY_ID=$(aws kms create-key \
  --description "xfusion-KMS-Key" \
  --key-usage ENCRYPT_DECRYPT \
  --customer-master-key-spec SYMMETRIC_DEFAULT \
  --query KeyMetadata.KeyId \
  --output text)

echo "Created Key ID: $KEY_ID"
```

#### Sample Output

```bash
Created Key ID: ac6dd134-bcaf-4220-892c-a7629fd19a0e
```

***

### Step 2: Create Alias for the Key

Create a user-friendly alias for easier reference:

```bash
aws kms create-alias \
  --alias-name alias/xfusion-KMS-Key \
  --target-key-id $KEY_ID
```

#### Sample Output

(No output indicates successful execution)

***

### Step 3: Encrypt the File

Encrypt the sensitive file and store the encrypted output:

```bash
aws kms encrypt \
  --key-id alias/xfusion-KMS-Key \
  --plaintext fileb:///root/SensitiveData.txt \
  --query CiphertextBlob \
  --output text | base64 --decode > /root/EncryptedData.bin
```

#### Output

*   Encrypted file created at:

    ```
    /root/EncryptedData.bin
    ```

***

### Step 4: Decrypt the File

Decrypt the encrypted file to verify correctness:

```bash
aws kms decrypt \
  --ciphertext-blob fileb:///root/EncryptedData.bin \
  --query Plaintext \
  --output text | base64 --decode > /root/DecryptedData.txt
```

#### Output

*   Decrypted file created at:

    ```
    /root/DecryptedData.txt
    ```

***

### Step 5: Verify Data Integrity

Compare the original and decrypted files:

```bash
diff /root/SensitiveData.txt /root/DecryptedData.txt && echo "Match successful"
```

#### Sample Output

```bash
Match successful
```

***

### Validation Criteria

The implementation meets the following requirements:

* A symmetric KMS key is created
* Alias `alias/xfusion-KMS-Key` is configured
* File is encrypted and stored as `EncryptedData.bin`
* Decryption successfully restores the original data
* Data integrity is verified

***

### Notes

* The `fileb://` prefix ensures binary-safe input/output handling
* AWS KMS returns base64-encoded ciphertext; decoding is required before saving
* Alias usage simplifies key management and avoids hardcoding Key IDs
* Ensure the correct AWS region is used during all operations

***

### Conclusion

This process demonstrates secure encryption and decryption of sensitive data using AWS KMS. The approach aligns with best practices for managing encryption keys and verifying data integrity in a controlled environment.

<figure><img src=".gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

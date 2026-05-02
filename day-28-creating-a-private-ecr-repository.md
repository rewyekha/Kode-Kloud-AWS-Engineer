# Day 28: Creating a Private ECR Repository

The Nautilus DevOps team has been tasked with setting up a containerized application. They need to create a private Amazon Elastic Container Registry (ECR) repository to store their Docker images. Once the repository is created, they will build a Docker image from a Dockerfile located on the `aws-client` host and push this image to the ECR repository. This process is essential for maintaining and deploying containerized applications in a streamlined manner.

Create a private ECR repository named `xfusion-ecr`. There is a Dockerfile under `/root/pyapp` directory on `aws-client` host, build a docker image using this Dockerfile and push the same to the newly created ECR repo, the image tag must be `latest`.

> **Note:** Run `showcreds` on the `aws-client` host to retrieve temporary AWS credentials, then configure the AWS CLI using `aws configure`.

**Notes:**

* Create the resources only in `us-east-1` region.
 ***

## AWS ECR – Creating Private Repository & Pushing Docker Image

### Task Overview

The objective of this task is to:

*   Create a **private ECR repository** named:

    ```
    xfusion-ecr
    ```
*   Build a Docker image from:

    ```
    /root/pyapp/Dockerfile
    ```
*   Tag the image as:

    ```
    latest
    ```
*   Push the image to AWS ECR in:

    ```
    us-east-1
    ```

***

## Step 1 – Authenticate Docker with AWS ECR

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 139373540961.dkr.ecr.us-east-1.amazonaws.com
```

#### Output

```bash
WARNING! Your credentials are stored unencrypted in '/root/.docker/config.json'.
Configure a credential helper to remove this warning. See
https://docs.docker.com/go/credential-store/

Login Succeeded
```

***

## Step 2 – Navigate to Application Directory

```bash
cd /root/pyapp
```

***

## Step 3 – Build Docker Image

```bash
docker build -t xfusion-ecr:latest .
```

#### Output

```bash
[+] Building 187.1s (9/9) FINISHED
 => [internal] load build definition from Dockerfile 0.1s
 => => transferring dockerfile: 164B
 => [internal] load metadata for docker.io/library/python:3.8-slim 122.1s
 => [1/4] FROM docker.io/library/python:3.8-slim ... 61.8s
 => [2/4] COPY . /app 0.1s
 => [3/4] WORKDIR /app 0.1s
 => [4/4] RUN pip install -r requirements.txt 2.5s
 => exporting to image 0.2s
 => => naming to docker.io/library/xfusion-ecr:latest
```

***

## Step 4 – Tag Docker Image for ECR

```bash
docker tag xfusion-ecr:latest 139373540961.dkr.ecr.us-east-1.amazonaws.com/xfusion-ecr:latest
```

***

## Step 5 – Verify Project Files

```bash
ls
```

#### Output

```bash
app.py  Dockerfile  requirements.txt
```

***

### Dockerfile

```bash
cat Dockerfile
```

#### Output

```bash
FROM python:3.8-slim
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

***

### app.py

```bash
cat app.py
```

#### Output

```bash
print("Hello, World!")
```

***

### requirements.txt

```bash
cat requirements.txt
```

#### Output

_(empty file)_

***

## Step 6 – Push Image to AWS ECR

```bash
docker push 139373540961.dkr.ecr.us-east-1.amazonaws.com/xfusion-ecr:latest
```

#### Output

```bash
The push refers to repository [139373540961.dkr.ecr.us-east-1.amazonaws.com/xfusion-ecr]

33690f00a8b8: Pushed
5f70bf18a086: Pushed
84ec9aef0828: Pushed
d2a2207b52a4: Pushed
5d2d143f3d7f: Pushed
c3772b569c3a: Pushed
8d853c8add5d: Pushed

latest: digest: sha256:3d9031f2f7555a0bd8e59615069f909b782cc102164ca8746d6faf568aa9d8f5 size: 1783
```

***

## Final Result Summary

### Successfully Completed

*  AWS ECR login successful
*  Docker image built successfully
*  Image tagged correctly for ECR
*  Image pushed to ECR repository
*  Repository: `xfusion-ecr`
*  Tag: `latest`

***

## Final Image Location

```
139373540961.dkr.ecr.us-east-1.amazonaws.com/xfusion-ecr:latest
```

***

## Conclusion

This task demonstrates a full container workflow:

```
Code (Python app)
   ↓
Docker Build
   ↓
Tag Image
   ↓
AWS ECR Push
   ↓
Private Container Registry
```

***

<figure><img src=".gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

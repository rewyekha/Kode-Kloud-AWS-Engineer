# Day 38: Deploying Containerized Applications with Amazon ECS

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

The Nautilus DevOps team is tasked with deploying a containerized application using Amazon's container services. They need to create a private Amazon Elastic Container Registry (ECR) to store their Docker images and use Amazon Elastic Container Service (ECS) to deploy the application. The process involves building a Docker image from a given Dockerfile, pushing it to the ECR, and then setting up an ECS cluster to run the application.

1. **Create a Private ECR Repository**:
   * Create a private ECR repository named `nautilus-ecr` to store Docker images.
2. **Build and Push Docker Image**:
   * Use the Dockerfile located at `/root/pyapp` on the `aws-client` host.
   * Build a Docker image using this Dockerfile.
   * Tag the image with `latest` tag.
   * Push the Docker image to the `nautilus-ecr` repository.
3. **Create and Configure ECS cluster**:
   * Create an ECS cluster named `nautilus-cluster` using the Fargate launch type.
4. **Create an ECS Task Definition**:
   * Define a task named `nautilus-taskdefinition` using the Docker image from the `nautilus-ecr` ECR repository.
   * Specify necessary CPU and memory resources.
5. **Deploy the Application Using ECS Service**:
   * Create a service named `nautilus-service` on the `nautilus-cluster` to run the task.
   * Ensure the service runs at least one task.

Use below given AWS Credentials: (You can run the `showcreds` command on `aws-client` host to retrieve these credentials)

`Notes:`

* Create the resources only in `us-east-1` region.

<figure><img src=".gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

---

> Portfolio: https://reyaskhan.me | GitHub: https://github.com/rewyekha

# Web Application Deployment on AWS ECS Fargate

### Project Overview

This documentation details the deployment steps for the web application on the AWS platform using ECS Fargate. The steps cover VPC setup, ECR repository creation, Docker image building, database configuration, secrets management, load balancing, and container deployment.

---

### Table of Contents

1. [Create a VPC](#step-1-create-a-vpc)
2. [Create a Repository in Elastic Container Registry (ECR)](#step-2-create-a-repository-in-elastic-container-registry-ecr)
3. [Build a Docker Image](#step-3-build-a-docker-image)
4. [Push the Docker Image into ECR](#step-4-push-the-docker-image-into-ecr)
5. [Set Up an RDS Database](#step-5-set-up-an-rds-database)
6. [Configure Secrets Manager](#step-6-configure-secrets-manager)
7. [Create a Target Group](#step-7-create-a-target-group)
8. [Create an Application Load Balancer](#step-8-create-an-application-load-balancer)
9. [Set Up IAM Roles and Policies](#step-9-set-up-iam-roles-and-policies)
10. [Create an ECS Cluster](#step-10-create-an-ecs-cluster)
11. [Create an ECS Task Definition](#step-11-create-an-ecs-task-definition)
12. [Create an ECS Service](#step-12-create-an-ecs-service)

---

## Deployment Steps

### Step 1: Create a VPC
1. **Virtual Private Cloud (VPC)**: A private network to host resources like EC2 instances and databases.
   - **Best Practice**: Segregate public and private subnets across availability zones for high availability.
2. **VPC Architecture**:
   - Two availability zones.
   - Six subnets: two public, two application (private), and two database (private).
3. **Public Subnets**: Host services exposed to the internet.
4. **NAT Gateway & Internet Gateway**: Allow internet access for private instances and the VPC.
5. **Security Groups**:
   - **Security Group 1**: Allows HTTP traffic (port 80) for the application load balancer.
   - **App Security Group**: Allows HTTP traffic (port 80) from the ALB to containers.
   - **Data Security Group**: Allows MySQL traffic (port 3306) for the RDS instance.

### Step 2: Create a Repository in Elastic Container Registry (ECR)
1. **Elastic Container Registry (ECR)**: Hosts Docker images for application containers.
2. **Repository_name**: A private repository to store the Docker image.

### Step 3: Build a Docker Image
1. **Docker**: Use Dockerfile instructions to build the Docker image.
2. **Docker Image Build Server**:
   - Use an EC2 instance for image building.(Same results can be achieved using the terminal in your local machine)
   - Assign an IAM role with the `AmazonSSMManagedInstanceCore` and `AmazonEC2InstanceProfileForImageBuilderECRContainerBuilds` policies.
3. **Build Process**:
   - Install Docker on EC2, download source code from GitHub, and build the image using the Dockerfile.

### Step 4: Push the Docker Image into ECR
1. **Push Commands**: Obtain commands from the ECR console.
2. **Authentication**: Authenticate Docker client with ECR.
3. **Push**: Push the Docker image to the Repository_name.

### Step 5: Set Up an RDS Database
1. **RDS Database**: Managed MySQL database for application data.
2. **Database Subnet Group**: Defines subnets for the database.
3. **MySQL Database Instance**: Single instance for the project (multi-AZ recommended for production).

### Step 6: Configure Secrets Manager
1. **Secrets Manager**: Securely stores database credentials with automatic rotation.
2. **Dynamic Credential Retrieval**: Application retrieves credentials from Secrets Manager.
3. **Lambda Function**: Rotates credentials periodically.
4. **Security Group**: Allow Lambda and RDS communication by configuring the Data Security Group.

### Step 7: Create a Target Group
1. **Target Group**: Holds container IP addresses.
2. **Configuration**:
   - Target type: IP addresses.
   - Protocol: HTTP, Port: 80.
   - Health check path: `/health.html`.

### Step 8: Create an Application Load Balancer
1. **Application Load Balancer**: Distributes traffic to containers.
2. **Configuration**:
   - Internet-facing, attached to the Richard Roast VPC, using public subnets.
   - Security Group:   Security Group.
   - Target Group: Richard Roast Target Group.

### Step 9: Set Up IAM Roles and Policies
1. **IAM Role for ECS Tasks**: Allows ECS containers to interact with ECR, CloudWatch, and Secrets Manager.
2. **Policies**:
   - `AmazonECSTaskExecutionRolePolicy`: For ECR access and CloudWatch logging.
   - `RichardRoastAllowReadDBSecretPolicy`: For Secrets Manager access.

### Step 10: Create an ECS Cluster
1. **ECS Cluster**: A logical grouping of containers.
2. **Cluster Type**: Fargate (serverless).

### Step 11: Create an ECS Task Definition
1. **Task Definition**: Specifies how containers are built and configured.
2. **Configuration**:
   - Launch type: AWS Fargate.
   - Memory and CPU: 0.5 vCPUs and 1 GB memory.
   - IAM Role: Richard Roast ECS task role.
   - Container settings:
     - Image URI from ECR.
     - Port mappings (Port 80 for HTTP).
   - Environmental Variables:
     - `DB_SERVER`, `DB_DATABASE`, `SECRET_NAME`, `AWS_REGION`.
   - Logging: CloudWatch.

### Step 12: Create an ECS Service
1. **ECS Service**: Deploys and manages containers in the ECS cluster.
2. **Configuration**:
   - Launch type: Fargate.
   - Service name: Richard Roast service.
   - Networking: Richard Roast VPC, app subnets, App Security Group.
   - Load Balancer:   and Target Group.
   - Auto Scaling: Enabled, with target CPU utilization at 70%.

**Access the Application**: Once the ECS service is active, the web application is accessible via the application load balancer’s DNS name.

---

Thank you..

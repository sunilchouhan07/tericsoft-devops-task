# Tericsoft DevOps Internship Task – Containerized Application Deployment

A containerized web application deployed on **AWS EC2 and Amazon ECS Fargate** using Docker. This project demonstrates cloud infrastructure provisioning, containerization, image management, networking, and application deployment.

## Project Overview

The objective of this project was to build a simple web application, containerize it using Docker, and deploy it on AWS.

The application is deployed using two approaches:

1. **Amazon EC2:** Running a Docker container on an Ubuntu-based EC2 instance.
2. **Amazon ECS Fargate:** Running the same containerized application using AWS-managed container infrastructure.

The application uses Nginx to serve static HTML and CSS content and is publicly accessible over the internet.

## Architecture

### 1. EC2 Deployment

```text
                 Internet
                    |
                    |
             EC2 Public IP
                    |
             Security Group
              TCP Port 8081
                    |
                    |
              Docker Engine
                    |
              Port Mapping
              8081 : 80
                    |
              Docker Container
                    |
                  Nginx
                    |
              HTML + CSS
```

### 2. ECS Fargate Deployment

```text
                Docker Image
                tericsoft-web:v1
                     |
                     |
              Amazon ECR
                     |
                     |
               ECS Cluster
                     |
                ECS Service
                     |
               Fargate Task
                     |
              Docker Container
                     |
                   Nginx
                     |
                 Port 80
                     |
                Public IP
                     |
                  Internet
```

**Networking:** ECS uses Fargate with `awsvpc` networking, a public subnet, an assigned public IP, and a security group allowing inbound TCP traffic on port 80. No Application Load Balancer or NAT Gateway is used.

## Technology Stack

| Technology | Purpose |
|---|---|
| AWS EC2 | Cloud compute infrastructure |
| AWS ECS | Container orchestration |
| AWS Fargate | Serverless container compute |
| Amazon ECR | Docker image registry |
| IAM | Access management |
| Ubuntu 24.04 LTS | Operating system |
| Docker | Application containerization |
| Nginx | Web server |
| HTML5 | Application structure |
| CSS3 | Application styling |
| Git & GitHub | Version control |

## Project Structure

```text
tericsoft-devops-task/
│
├── index.html
├── style.css
├── Dockerfile
├── .dockerignore
└── README.md
```

## AWS Infrastructure

### AWS Region

`us-west-2` – US West (Oregon)

### EC2 Configuration

| Configuration | Value |
|---|---|
| Operating System | Ubuntu Server 24.04 LTS |
| Instance Type | t3.micro |
| Storage | 8 GiB gp3 |
| Container Runtime | Docker |
| Web Server | Nginx |
| Host Port | 8081 |
| Container Port | 80 |

### EC2 Security Group

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH | TCP | 22 | My IP |
| Custom TCP | TCP | 8081 | 0.0.0.0/0 |

### ECS Fargate Configuration

| Configuration | Value |
|---|---|
| Cluster | tericsoft-cluster |
| Service | tericsoft-service |
| Task Definition | tericsoft-task |
| Launch Type | AWS Fargate |
| CPU | 0.25 vCPU |
| Memory | 0.5 GB |
| Desired Tasks | 1 |
| Container Port | 80 |
| Host Port | 80 |
| Network Mode | awsvpc |
| Public IP | Enabled |
| Load Balancer | Not configured |

### ECS Security Group

| Type | Protocol | Port | Source |
|---|---|---|---|
| Custom TCP | TCP | 80 | 0.0.0.0/0 |

## Dockerfile

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
COPY style.css /usr/share/nginx/html/style.css

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Docker Configuration Explanation

- `FROM nginx:alpine`: Uses a lightweight Nginx base image.
- `COPY`: Copies application files into the Nginx web root.
- `EXPOSE 80`: Documents the container's listening port.
- `CMD`: Runs Nginx in the foreground.

## Deployment 1: AWS EC2

### 1. Connect to EC2

```bash
ssh -i tericsoft-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

### 2. Install Docker

```bash
sudo apt update
sudo apt install docker.io -y
```

Enable Docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Configure Docker permissions:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Verify installation:

```bash
docker --version
docker ps
```

### 3. Build Docker Image

Navigate to the project directory:

```bash
cd ~/tericsoft-devops-task
```

Build the image:

```bash
docker build -t tericsoft-web:v1 .
```

Verify:

```bash
docker images
```

### 4. Run Docker Container

```bash
docker run -d \
  --name tericsoft-app \
  -p 8081:80 \
  tericsoft-web:v1
```

Verify the running container:

```bash
docker ps
```

Test locally:

```bash
curl http://localhost:8081
```

### 5. Access Application

Open the following URL:

```text
http://YOUR_EC2_PUBLIC_IP:8081
```

The application should be accessible publicly through the EC2 public IPv4 address.

## Deployment 2: Amazon ECR

Amazon ECR is used to store the Docker image that ECS Fargate pulls during deployment.

### 1. Create ECR Repository

Repository name:

```text
tericsoft-web
```

Repository type: Private

### 2. Configure EC2 IAM Role

Created an EC2 IAM role with the AWS managed policy:

```text
AmazonEC2ContainerRegistryPowerUser
```

This allows the EC2 instance to authenticate with ECR and push Docker images.

### 3. Authenticate Docker with ECR

Set the AWS region and account ID:

```bash
REGION=us-west-2
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
```

Authenticate:

```bash
aws ecr get-login-password --region $REGION | \
docker login \
--username AWS \
--password-stdin \
$ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com
```

### 4. Tag Docker Image

```bash
docker tag tericsoft-web:v1 \
$ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/tericsoft-web:v1
```

### 5. Push Image to ECR

```bash
docker push \
$ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/tericsoft-web:v1
```

Verify the uploaded image in the Amazon ECR console.

## Deployment 3: Amazon ECS Fargate

### 1. Create ECS Cluster

Cluster name:

```text
tericsoft-cluster
```

Infrastructure: AWS Fargate.

No EC2 capacity provider is required.

### 2. Create Task Execution Role

Created an IAM role for ECS task execution with:

```text
AmazonECSTaskExecutionRolePolicy
```

This allows ECS to retrieve container images from ECR and initialize container logging.

### 3. Create Task Definition

| Setting | Value |
|---|---|
| Family | tericsoft-task |
| Launch Type | Fargate |
| Operating System | Linux |
| Architecture | X86_64 |
| CPU | 0.25 vCPU |
| Memory | 0.5 GB |
| Execution Role | tericsoftECSTaskExecutionRole |

Container configuration:

**Container name:**

```text
tericsoft-web
```

**Image URI:**

```text
ACCOUNT_ID.dkr.ecr.us-west-2.amazonaws.com/tericsoft-web:v1
```

**Port Mapping:**

```text
Container Port: 80
Protocol: TCP
```

### 4. Create ECS Service

| Setting | Value |
|---|---|
| Service Name | tericsoft-service |
| Cluster | tericsoft-cluster |
| Launch Type | Fargate |
| Desired Tasks | 1 |
| Deployment Type | Rolling Update |

### 5. Configure Networking

- Selected the default VPC.
- Selected a public subnet.
- Enabled public IP assignment.
- Attached the ECS security group.
- Allowed inbound TCP traffic on port 80.
- Did not configure an Application Load Balancer or NAT Gateway.

### 6. Verify ECS Deployment

Navigate to:

```text
Amazon ECS
    → Clusters
    → tericsoft-cluster
    → Services
    → tericsoft-service
```

Verify:

```text
Desired Tasks: 1
Running Tasks: 1
Pending Tasks: 0
```

Retrieve the public IP from the running Fargate task.

Access the application:

```text
http://YOUR_ECS_PUBLIC_IP/
```

The same containerized application should be accessible through ECS Fargate.

## EC2 vs ECS Deployment

| Feature | EC2 | ECS Fargate |
|---|---|---|
| Compute | EC2 instance | Managed Fargate compute |
| Container Runtime | Docker | Docker |
| Image Source | Local Docker image | Amazon ECR |
| Deployment | Docker CLI | ECS Service |
| Public Port | 8081 | 80 |
| Infrastructure Management | User-managed EC2 | AWS-managed compute |
| Orchestration | Manual | ECS-managed |

## Troubleshooting and Problems Solved

### 1. Docker Permission Issue

**Problem:** Docker commands required sudo privileges.

**Solution:** Added the Ubuntu user to the Docker group using `usermod -aG docker $USER` and refreshed group membership.

### 2. Public Application Access

**Problem:** Application needed to be accessible externally.

**Solution:** Configured the EC2 security group to allow inbound TCP port 8081 and mapped host port 8081 to container port 80.

### 3. ECR Authentication Failure

**Problem:** Docker push failed with `no basic auth credentials`.

**Solution:** Attached an IAM role with ECR permissions to EC2 and authenticated Docker using `aws ecr get-login-password`.

### 4. ECS Image Deployment

**Problem:** ECS required access to the private ECR repository.

**Solution:** Configured an ECS task execution role with `AmazonECSTaskExecutionRolePolicy` and used the ECR image URI in the task definition.

## Verification Checklist

### EC2

- [x] EC2 instance provisioned.
- [x] Docker installed and configured.
- [x] Docker image built.
- [x] Container started successfully.
- [x] Port mapping configured.
- [x] Security group configured.
- [x] Application accessible publicly.

### Amazon ECR

- [x] Private ECR repository created.
- [x] EC2 IAM permissions configured.
- [x] Docker authenticated with ECR.
- [x] Docker image pushed successfully.

### Amazon ECS

- [x] ECS Fargate cluster created.
- [x] Task execution IAM role configured.
- [x] Task definition created.
- [x] ECS service deployed.
- [x] Fargate task reached RUNNING state.
- [x] Public IP assigned.
- [x] Application accessible publicly.

## Cost Management

AWS resources may incur charges depending on account eligibility, resource configuration, and usage.

To minimize unnecessary expenses:

- Use eligible instance configurations where available.
- Monitor EC2 and Fargate usage.
- Avoid unnecessary NAT Gateway and load balancer resources.
- Set ECS service desired count to zero after the demonstration.
- Delete the ECS service and cluster when no longer required.
- Remove unused EBS volumes and public IPv4 addresses.
- Review AWS Billing and Cost Explorer regularly.

## Author

**Sunil Chouhan**

GitHub: https://github.com/sunilchouhan07

## Internship Assessment

**Organization:** Tericsoft Technologies Pvt. Ltd.

**Task:** DevOps Internship – Containerized App Deployment

This project demonstrates hands-on experience with AWS EC2, Docker, Amazon ECR, Amazon ECS Fargate, IAM, Linux, networking, and cloud-based application deployment.

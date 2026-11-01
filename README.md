# Tericsoft DevOps Internship Task – Containerized App Deployment

A simple web application containerized using Docker and deployed on an AWS EC2 instance. This project demonstrates containerization, cloud infrastructure provisioning, networking, and public application access.

## Project Overview

The objective of this project is to deploy a web application inside a Docker container on AWS EC2 and make it publicly accessible through a custom port.

The application uses Nginx as the web server to serve static HTML and CSS content.

## Architecture

```text
             Internet
                 |
                 |
          AWS EC2 Public IP
                 |
          Security Group
          Inbound TCP 8081
                 |
                 |
          Docker Container
                 |
          Host Port: 8081
                 |
        Container Port: 80
                 |
              Nginx
                 |
           HTML + CSS
```

## Technology Stack

| Technology | Purpose |
|---|---|
| AWS EC2 | Cloud compute infrastructure |
| Ubuntu 24.04 LTS | Operating system |
| Docker | Application containerization |
| Nginx | Web server |
| HTML5 | Application structure |
| CSS3 | Application styling |
| Git & GitHub | Version control |

## AWS Infrastructure

- **Cloud Provider:** Amazon Web Services
- **Region:** US West (Oregon) – `us-west-2`
- **Compute:** EC2
- **Operating System:** Ubuntu Server 24.04 LTS
- **Instance Type:** t3.micro (subject to account eligibility)
- **Storage:** 8 GiB gp3
- **Networking:** Default VPC with public IPv4 address

### Security Group Configuration

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH | TCP | 22 | My IP |
| Custom TCP | TCP | 8081 | 0.0.0.0/0 |

Port 8081 allows external users to access the deployed web application.

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

## Deployment Guide

### 1. Connect to EC2

Connect to the Ubuntu instance using SSH:

```bash
ssh -i tericsoft-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

### 2. Install Docker

Update the package index:

```bash
sudo apt update
```

Install Docker:

```bash
sudo apt install docker.io -y
```

Enable and start Docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Add the current user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

Apply group membership:

```bash
newgrp docker
```

Verify installation:

```bash
docker --version
docker ps
```

### 3. Build the Docker Image

Clone the repository or navigate to the project directory:

```bash
cd tericsoft-devops-task
```

Build the Docker image:

```bash
docker build -t tericsoft-web:v1 .
```

Verify the image:

```bash
docker images
```

### 4. Run the Docker Container

Start the container:

```bash
docker run -d \
  --name tericsoft-app \
  -p 8081:80 \
  tericsoft-web:v1
```

**Port Mapping:**

- Host port: `8081`
- Container port: `80`
- Nginx listens on port `80` inside the container.
- Docker publishes host port `8081` and forwards traffic to container port `80`.

### 5. Verify Container Status

```bash
docker ps
```

Check application response locally:

```bash
curl http://localhost:8081
```

### 6. Access the Application

Open the following URL in your browser:

```text
http://YOUR_EC2_PUBLIC_IP:8081
```

The application should be accessible publicly over the internet.

## Dockerfile

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
COPY style.css /usr/share/nginx/html/style.css

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

## Docker Commands Used

| Command | Description |
|---|---|
| `docker build -t tericsoft-web:v1 .` | Build Docker image |
| `docker images` | List available images |
| `docker run -d` | Run container in detached mode |
| `docker ps` | List running containers |
| `docker logs tericsoft-app` | View container logs |
| `docker stop tericsoft-app` | Stop container |
| `docker rm tericsoft-app` | Remove container |

## Deployment Status

### EC2 Deployment

- [x] EC2 instance provisioned
- [x] Docker installed and configured
- [x] Application containerized
- [x] Docker image built successfully
- [x] Container running successfully
- [x] Port mapping configured
- [x] Security group configured
- [x] Public application access verified
- [x] Source code pushed to GitHub

### ECS Bonus Deployment

- [ ] Amazon ECR repository
- [ ] Docker image pushed to ECR
- [ ] ECS cluster created
- [ ] Task definition configured
- [ ] ECS service deployed
- [ ] Public application access verified

## Cost Management

This project uses AWS resources that may incur charges depending on account eligibility and usage.

To avoid unnecessary expenses:

- Stop or terminate the EC2 instance after completing the assessment.
- Delete unused EBS volumes and other billable resources.
- Stop or delete ECS services after testing.
- Remove unused public IP addresses and other billable resources.
- Review AWS Billing regularly.

## Author

**Sunil Chouhan**

- GitHub: [https://github.com/sunilchouhan07](https://github.com/sunilchouhan07)

## Internship Assessment

**Organization:** Tericsoft Technologies Pvt. Ltd.

**Task:** DevOps Internship – Containerized App Deployment

The project demonstrates practical experience with AWS EC2, Docker, Linux, container networking, and application deployment.

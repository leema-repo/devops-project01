# Brain Tasks App – DevOps Deployment Project

## Project Overview

This project demonstrates a complete **DevOps CI/CD pipeline** for deploying a production-ready web application using AWS services.

The application is containerized using **Docker**, stored in **Amazon ECR**, and deployed to a **Kubernetes cluster running on Amazon EKS**. The deployment process is automated using **AWS CodePipeline** and **AWS CodeBuild**.

This project focuses on **deployment and infrastructure automation**, since the repository already contains the compiled **dist build files**.

---

## Architecture

GitHub
↓
AWS CodePipeline
↓
AWS CodeBuild
↓
Docker Image Build
↓
Amazon ECR
↓
Amazon EKS
↓
Kubernetes Pods
↓
AWS LoadBalancer
↓
Application Access

---

## Tools & Technologies Used

* AWS EC2 (Ubuntu 22.04)
* Git & GitHub
* Docker
* Amazon ECR
* Amazon EKS
* Kubernetes
* AWS CodeBuild
* AWS CodePipeline
* AWS CloudWatch
* Nginx

---

## Pre-Requisites

* AWS Account
* Ubuntu 22.04 EC2 Instance
* AWS CLI Installed
* kubectl Installed
* eksctl Installed
* Docker Installed
* Git Installed

---

## Clone the Repository

```bash
git clone https://github.com/YOUR-USERNAME/devops-project01.git
cd devops-project01
```

---

## Docker Setup

### Dockerfile

```dockerfile
FROM nginx:alpine
COPY dist/ /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Build Docker Image

```bash
docker build -t brain-tasks-app .
```

### Run Container Locally

```bash
docker run -p 3000:80 brain-tasks-app
```

Access application locally:

```
http://localhost:3000
```

---

## Push Docker Image to Amazon ECR

### Create ECR Repository

```bash
aws ecr create-repository --repository-name brain-tasks-app --region us-west-2
```

### Login to ECR

```bash
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-west-2.amazonaws.com
```

### Tag Docker Image

```bash
docker tag brain-tasks-app:latest <account-id>.dkr.ecr.us-west-2.amazonaws.com/brain-tasks-app:latest
```

### Push Image to ECR

```bash
docker push <account-id>.dkr.ecr.us-west-2.amazonaws.com/brain-tasks-app:latest
```

---

## Create Amazon EKS Cluster

```bash
eksctl create cluster \
--name devops-cluster \
--region us-west-2 \
--nodegroup-name devops-nodes \
--node-type t3.medium \
--nodes 2
```

### Verify Cluster

```bash
kubectl get nodes
```

---

## Kubernetes Deployment

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: brain-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: brain-app
  template:
    metadata:
      labels:
        app: brain-app
    spec:
      containers:
      - name: brain-app
        image: <account-id>.dkr.ecr.us-west-2.amazonaws.com/brain-tasks-app:latest
        ports:
        - containerPort: 80
```

---

### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: brain-app-service
spec:
  type: LoadBalancer
  selector:
    app: brain-app
  ports:
  - port: 80
    targetPort: 80
```

---

## Deploy Application to Kubernetes

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### Verify Deployment

```bash
kubectl get pods
kubectl get svc
```

---

## CI/CD Pipeline (AWS CodePipeline)

The CI/CD pipeline automates the build and deployment process.

### Pipeline Stages

**Source Stage**

* GitHub repository connected to CodePipeline

**Build Stage**

* AWS CodeBuild builds Docker image
* Image pushed to Amazon ECR

**Deploy Stage**

* Kubernetes deployment executed on Amazon EKS

---

## Monitoring

Application monitoring and pipeline logs are handled using **AWS CloudWatch Logs**.

CloudWatch tracks:

* CodeBuild build logs
* CodePipeline execution logs
* Kubernetes deployment events

---

## Application URL

```
http://a02ea846210d145bdb3e34d01ec17a7b-1530721649.us-west-2.elb.amazonaws.com
```

---

## Load Balancer ARN

Replace with the actual ARN from AWS console:

```
arn:aws:elasticloadbalancing:us-west-2:YOUR_ACCOUNT_ID:loadbalancer/net/k8s-brain-app-service/XXXXXXXX
```

---

## Screenshots

The following screenshots are included in the document:

* GitHub Repository
* AWS CodePipeline Pipeline
* AWS CodeBuild Build Logs
* Amazon ECR Repository
* Amazon EKS Cluster
* Kubernetes Pods
* Kubernetes Services
* Application running in browser

# 📘 Task #11 - Blue/Green Deployment for Strapi using ECS Fargate & CodeDeploy

## ✅ Objective
Set up a **Blue/Green deployment pipeline** for the Strapi app on AWS ECS Fargate using **CodeDeploy**, **ALB**, and **Terraform**.

---

## ⚙️ Infrastructure Setup (via Terraform)

### 1. ECS Cluster & Fargate Service

- Created ECS cluster: `strapi-varun-cluster`
- ECS service runs on **Fargate**
- Service uses **CodeDeploy** deployment controller

### 2. Application Load Balancer (ALB)

- ALB with **2 Target Groups**:
  - **Blue** for currently running version
  - **Green** for new version during deployment
- ALB Listener on **port 80** with default forwarding to the Blue target group

### 3. Security Groups

- **ALB Security Group**: Allows inbound on ports **80 (HTTP)** and **443 (HTTPS)**
- **ECS Security Group**: Allows traffic **from ALB** on port **1337** (Strapi port)

### 4. ECS Task Definition

- Created ECS Task Definition with:
  - Placeholder Docker image
  - Environment variables for DB connection
  - Logging to CloudWatch
- Will be dynamically updated during deployment with the new image tag

---

## 🚀 CodeDeploy Configuration

### 1. CodeDeploy Application

- Created ECS-specific CodeDeploy app: `strapi-codedeploy-app`

### 2. Deployment Group

- Name: `strapi-bluegreen-deployment-group`
- **Deployment Type**: `BLUE_GREEN`
- **Deployment Option**: `WITH_TRAFFIC_CONTROL`
- Target groups: `strapi-tg-blue`, `strapi-tg-green`
- Listener: ALB listener on port 80
- **Strategy**: `CodeDeployDefault.ECSCanary10Percent5Minutes`
  - 10% traffic for 5 minutes, then shift remaining
- **Rollback**: Enabled on failure
- **Termination**: Old (blue) version terminated after success

---

## 🔐 IAM Roles

- **ecs_task_execution_role**:
  - Allows ECS tasks to pull from ECR & log to CloudWatch
- **codedeploy_role**:
  - Allows CodeDeploy to perform ECS deployment actions

---

## 🧪 Deployment Testing

- Verified app deployment through:
  - Green target group receiving new deployment
  - ALB switching traffic after health checks pass
  - Blue target group terminated post-success
- API verified at:

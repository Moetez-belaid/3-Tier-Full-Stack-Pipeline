# Three Tier Full Stack Project

[space for screenshot]

**Author:** Shubham Mukherjee  
**DevOps Focus:** CI/CD with Jenkins, Docker, SonarQube, Trivy, and AWS EKS  
**GitHub Repository:** [3-Tier-Full-Stack](https://github.com/ShubhamStunner/3-Tier-Full-Stack.git)

[space for screenshot]

## Table of Contents
1. [Introduction](#introduction)  
2. [Project Overview](#project-overview)  
3. [Architecture](#architecture)  
4. [Environments](#environments)  
5. [Environment Variables Setup](#environment-variables-setup)  
6. [Phase 1: Local Deployment](#phase-1-local-deployment)  
   - [Step 1: Install Node and Clone Repo](#step-1-install-node-and-clone-repo)  
   - [Step 2: Create a .env File](#step-2-create-a-env-file)  
   - [Step 3: Install Dependencies and Run App](#step-3-install-dependencies-and-run-app)  
7. [Phase 2: Development Environment Deployment](#phase-2-development-environment-deployment)  
   - [Step 1: Launch EC2 and Install Jenkins](#step-1-launch-ec2-and-install-jenkins)  
   - [Step 2: Install Docker and Trivy](#step-2-install-docker-and-trivy)  
   - [Step 3: Set Up SonarQube](#step-3-set-up-sonarqube)  
   - [Step 4: Configure Jenkins](#step-4-configure-jenkins)  
   - [Step 5: Create a Jenkins Pipeline](#step-5-create-a-jenkins-pipeline)  
8. [Phase 3: Production Environment Deployment](#phase-3-production-environment-deployment)  
   - [Step 1: Install AWS CLI, eksctl, and kubectl](#step-1-install-aws-cli-eksctl-and-kubectl)  
   - [Step 2: Configure AWS](#step-2-configure-aws)  
   - [Step 3: Create EKS Cluster & Node Group](#step-3-create-eks-cluster--node-group)  
   - [Step 4: Kubernetes Permissions (ServiceAccount, Roles, Bindings)](#step-4-kubernetes-permissions-serviceaccount-roles-bindings)  
   - [Step 5: Encode Environment Variables and Deploy](#step-5-encode-environment-variables-and-deploy)  
9. [Key Takeaways](#key-takeaways)  
10. [Acknowledgement](#acknowledgement)  
11. [Conclusion](#conclusion)  

---

## Introduction

[space for screenshot]

The **Three Tier Full Stack Project** is a comprehensive demonstration of how to build, test, secure, and deploy a multi-tiered web application. It uses:
- **Jenkins** for Continuous Integration (CI) and managing the build pipeline.  
- **Docker** to containerize the application for consistent environments.  
- **SonarQube** for code quality analysis.  
- **Trivy** for security scanning of both filesystems and container images.  
- **AWS EKS** for container orchestration, enabling scalable and fault-tolerant production deployments.

---

## Project Overview

[space for screenshot]

1. **Jenkins Setup**: Installs and configures Jenkins to automatically pull code from GitHub, run tests, build and scan Docker images, then push them to a registry.  
2. **Dockerization**: Ensures environment consistency by packaging the application into Docker containers.  
3. **SonarQube Analysis**: Continuously checks code for bugs and potential vulnerabilities.  
4. **Trivy Scans**: Adds another layer of security by scanning for vulnerabilities in both local files and Docker images.  
5. **AWS EKS**: Provides a fully managed Kubernetes cluster for production-ready deployments.

---

## Architecture

[space for screenshot]

A visual outline of the pipeline and architecture:





---

## Environments

[space for screenshot]

The project separates the lifecycle of the application into three main environments for better manageability:

1. **Local Environment**  
   - **Purpose**: Initial coding, quick testing, and debugging.  
   - **Workflow**:  
     1. Developer clones the repository.  
     2. Runs the app locally with Node.js.  
     3. Validates changes before committing.

2. **Development Environment**  
   - **Purpose**: Automated testing, code analysis, and container builds.  
   - **Workflow**:  
     1. Jenkins fetches the repository from GitHub.  
     2. Performs unit tests (npm test).  
     3. Runs SonarQube and Trivy scans.  
     4. Builds Docker image and pushes it to the Dev server (local Docker run).

3. **Production Environment**  
   - **Purpose**: Scalable and secure final deployment on AWS EKS.  
   - **Workflow**:  
     1. All previous steps (test, scan, build) re-run for final verification.  
     2. Docker image pushed to a production registry.  
     3. Deployed to an EKS cluster, automatically scalable with Kubernetes.

---

## Environment Variables Setup

[space for screenshot]

Using environment variables keeps sensitive data, such as API keys and credentials, out of source code. **Always** protect these values and avoid committing them to version control.

1. **Cloudinary**:  
   - `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_KEY`, `CLOUDINARY_SECRET`
2. **Mapbox**:  
   - `MAPBOX_TOKEN`
3. **MongoDB Atlas**:  
   - `DB_URL`  
4. **Application SECRET**:  
   - `SECRET`  

Example `.env` file:
```bash
CLOUDINARY_CLOUD_NAME=dj0lkx4sa
CLOUDINARY_KEY=686681518851598
CLOUDINARY_SECRET=hSGeIYLq2FtaokM80E4tlGnoVYY
MAPBOX_TOKEN=sk.eyJ1Ijoic2h1YmhhbTIx...
DB_URL="mongodb+srv://<username>:<password>@<cluster_url>/..."
SECRET=shubham




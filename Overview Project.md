```markdown
# Three Tier Full Stack Project

**Author:** Shubham Mukherjee  
**DevOps Focus:** CI/CD with Jenkins, Docker, SonarQube, Trivy, and AWS EKS  
**GitHub Repository:** [3-Tier-Full-Stack](https://github.com/ShubhamStunner/3-Tier-Full-Stack.git)

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
The **Three Tier Full Stack Project** is a comprehensive demonstration of how to build, test, secure, and deploy a multi-tiered web application. It uses:
- **Jenkins** for Continuous Integration (CI) and managing the build pipeline.  
- **Docker** to containerize the application for consistent environments.  
- **SonarQube** for code quality analysis.  
- **Trivy** for security scanning of both filesystems and container images.  
- **AWS EKS** for container orchestration, enabling scalable and fault-tolerant production deployments.

---

## Project Overview
1. **Jenkins Setup**: Installs and configures Jenkins to automatically pull code from GitHub, run tests, build and scan Docker images, then push them to a registry.  
2. **Dockerization**: Ensures environment consistency by packaging the application into Docker containers.  
3. **SonarQube Analysis**: Continuously checks code for bugs and potential vulnerabilities.  
4. **Trivy Scans**: Adds another layer of security by scanning for vulnerabilities in both local files and Docker images.  
5. **AWS EKS**: Provides a fully managed Kubernetes cluster for production-ready deployments.

---

## Architecture
A visual outline of the pipeline and architecture:

```
       ┌─────────┐
       │GitHub   │
       └───┬─────┘
           │        Code (main branch)
       ┌───▼──────────────────────────────────────────────┐
       │ Jenkins (CI Server)                             │
       │  * Pull latest code                              │
       │  * Run tests & SonarQube analysis               │
       │  * Build & tag Docker image                     │
       │  * Trivy scan                                   │
       │  * Push to registry                             │
       │  * Deploy to Docker (Dev)                       │
       │  * Deploy to EKS (Prod)                         │
       └───┬──────────────────────────────────────────────┘
           │
           │  Docker Image
           │
┌──────────▼───────────┐             ┌─────────────────────┐
│ Development Server   │             │ Production: AWS EKS  │
│ (Docker container)   │             │ * Managed Kubernetes │
│ Exposed on port 3000 │             │ * Highly Scalable    │
└──────────────────────┘             └─────────────────────┘
```

---

## Environments
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
```

> **Note**: This file should be included in your `.gitignore` to prevent accidental pushes.

---

## Phase 1: Local Deployment

### Step 1: Install Node and Clone Repo
1. **SSH into your Ubuntu machine**:  
   ```bash
   ssh -i <your_key.pem> ubuntu@<VM_IP>
   ```
2. **Install NVM and Node.js**:  
   ```bash
   # Install NVM
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
   
   # Load NVM commands in the current session
   export NVM_DIR="$HOME/.nvm"
   [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
   [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"

   # Install a specific version of Node
   nvm install 21
   node -v
   npm -v
   ```
3. **Clone the repository**:
   ```bash
   git clone https://github.com/ShubhamStunner/3-Tier-Full-Stack.git
   cd 3-Tier-Full-Stack
   ```

### Step 2: Create a `.env` File
Create a file in the project root directory to store sensitive keys:
```bash
vi .env
# Paste your keys:
CLOUDINARY_CLOUD_NAME=...
CLOUDINARY_KEY=...
CLOUDINARY_SECRET=...
MAPBOX_TOKEN=...
DB_URL=...
SECRET=...
```

### Step 3: Install Dependencies and Run App
1. **Install the project dependencies**:
   ```bash
   npm install
   ```
2. **Start the application**:
   ```bash
   npm start
   ```
3. **Access the application** in your browser at `http://<VM_IP>:3000`.

---

## Phase 2: Development Environment Deployment

### Step 1: Launch EC2 and Install Jenkins
1. **Create/Launch an EC2 instance** (for example, `t2.large`, Ubuntu 20.04).  
2. **SSH into the new instance**:
   ```bash
   ssh -i <your_key.pem> ubuntu@<jenkins_ec2_ip>
   ```
3. **Install Java (required by Jenkins)**:
   ```bash
   sudo apt update
   sudo apt install openjdk-17-jre-headless -y
   ```
4. **Add Jenkins apt repository and key**:
   ```bash
   sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
     https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

   echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
     https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
     /etc/apt/sources.list.d/jenkins.list > /dev/null
   ```
5. **Install Jenkins**:
   ```bash
   sudo apt-get update
   sudo apt-get install jenkins -y
   ```
6. **Open Jenkins in your browser** at `http://<jenkins_ec2_ip>:8080`.  
   - Use the initial admin password from `/var/lib/jenkins/secrets/initialAdminPassword` to unlock Jenkins.

### Step 2: Install Docker and Trivy
1. **Docker**:
   ```bash
   sudo apt-get install docker.io -y
   sudo chmod 666 /var/run/docker.sock  # Allows non-root user to run Docker
   ```
2. **Trivy**:  
   ```bash
   sudo apt-get install wget apt-transport-https gnupg lsb-release -y
   wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
   echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
   sudo apt-get update
   sudo apt-get install trivy -y
   ```

### Step 3: Set Up SonarQube
1. **Create another EC2 instance** (e.g., `t2.medium`) for SonarQube.  
2. **SSH into it** and install Docker (as above).  
3. **Run SonarQube container**:
   ```bash
   docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
   ```
4. **Access SonarQube** at `http://<sonarqube_ec2_ip>:9000`. Default credentials are `admin / admin`.

### Step 4: Configure Jenkins
1. **Install required plugins** (Manage Jenkins → Manage Plugins → Available tab):
   - Docker, NodeJS, SonarQube Scanner, Docker Pipeline, Kubernetes, Kubernetes CLI.
2. **NodeJS Plugin** setup (Manage Jenkins → Global Tool Configuration):
   - Add NodeJS installation (e.g., Node 21).
3. **SonarQube Scanner** setup (Manage Jenkins → Global Tool Configuration):
   - Add SonarQube Scanner (give it a name, e.g., `sonar-scanner`).  
   - In Manage Jenkins → Configure System → SonarQube servers, add the SonarQube server URL and authentication token.
4. **Docker Plugin** (Manage Jenkins → Global Tool Configuration):
   - Add Docker installation if you want Jenkins to handle Docker installation automatically (optional).
5. **Credentials**:
   - Add your GitHub credentials, Docker Registry credentials, SonarQube token, and any other secrets to Jenkins Credentials.

### Step 5: Create a Jenkins Pipeline
1. **Create a new Pipeline job** (e.g., `Dev-env-3tier`) under Jenkins → New Item.  
2. **Use the following example Jenkinsfile** (or place it directly in the Pipeline Script section):

```groovy
pipeline {
    agent any
    
    tools {
        nodejs 'node21'  // The name you gave in Global Tool Configuration
    }
    
    environment {
        SCANNER_HOME = tool "sonar-scanner" // The name you gave to the SonarQube Scanner
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'git-cred',    // Jenkins credential ID
                    url: 'https://github.com/Shubham-Stunner/3-Tier-Full-Stack.git'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        
        stage('Unit Test') {
            steps {
                sh "npm test"
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                // This step uses the SonarQube plugin to set up the environment
                withSonarQubeEnv('sonar') {
                    sh """
                        \$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=Campground \
                        -Dsonar.projectName=Campground
                    """
                }
            }
        }
        
        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred1', toolName: 'docker') {
                        sh "docker build -t stunnershubham/camp:latest ."
                    }
                }
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image-scan-report.html stunnershubham/camp:latest"
            }
        }
        
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred1', toolName: 'docker') {
                        sh "docker push stunnershubham/camp:latest"
                    }
                }
            }
        }
        
        stage('Docker Deploy to Dev') {
            steps {
                script {
                    // Runs the container on the Jenkins server (or any machine with Docker)
                    sh "docker run -d -p 3000:3000 stunnershubham/camp:latest"
                }
            }
        }
    }
}
```

> **Note**: Make sure you replace any credential IDs or Docker image names according to your own setup.

---

## Phase 3: Production Environment Deployment

### Step 1: Install AWS CLI, eksctl, and kubectl
1. **AWS CLI**:
   ```bash
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   sudo apt install unzip -y
   unzip awscliv2.zip
   sudo ./aws/install
   ```
2. **kubectl**:
   ```bash
   curl -o kubectl \
     https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
   chmod +x ./kubectl
   sudo mv ./kubectl /usr/local/bin
   kubectl version --short --client
   ```
3. **eksctl**:
   ```bash
   curl --silent --location \
     "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
     | tar xz -C /tmp
   sudo mv /tmp/eksctl /usr/local/bin
   eksctl version
   ```

### Step 2: Configure AWS
1. **Create a new IAM user** from the AWS console, assigning the following policies:
   - `AmazonEC2FullAccess`
   - `AmazonEKS_CNI_Policy`
   - `AmazonEKSClusterPolicy`
   - `AmazonEKSWorkerNodePolicy`
   - `AWSCloudFormationFullAccess`
   - `IAMFullAccess`
   - An **inline policy** allowing `"eks:*"` on `"*"`.
2. **Obtain the Access Key and Secret Key** for this user and run:
   ```bash
   aws configure
   # Provide Access Key, Secret Key, Region, and default output
   ```

### Step 3: Create EKS Cluster & Node Group
1. **Create EKS cluster** (control plane only at first):
   ```bash
   eksctl create cluster --name=EKS-1 \
     --region=ap-south-1 \
     --zones=ap-south-1a,ap-south-1b \
     --without-nodegroup
   ```
2. **Associate IAM OIDC** (required for assigning IAM roles to pods):
   ```bash
   eksctl utils associate-iam-oidc-provider \
     --region ap-south-1 \
     --cluster EKS-1 \
     --approve
   ```
3. **Create a node group**:
   ```bash
   eksctl create nodegroup --cluster=EKS-1 \
     --region=ap-south-1 \
     --name=node2 \
     --node-type=t3.medium \
     --nodes=3 \
     --nodes-min=2 \
     --nodes-max=3 \
     --node-volume-size=20 \
     --ssh-access \
     --ssh-public-key=<your-ssh-key> \
     --managed \
     --asg-access \
     --external-dns-access \
     --full-ecr-access \
     --appmesh-access \
     --alb-ingress-access
   ```

### Step 4: Kubernetes Permissions (ServiceAccount, Roles, Bindings)
To let Jenkins manipulate Kubernetes resources in the `webapps` namespace, you must create appropriate RBAC objects.

1. **Create ServiceAccount** (`svc.yaml`):
   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: jenkins
     namespace: webapps
   ```
   ```bash
   kubectl apply -f svc.yaml
   ```

2. **Create a Role** (`role.yaml`):
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     name: app-role
     namespace: webapps
   rules:
     - apiGroups: ["", "apps", "autoscaling", "batch", "extensions", "policy", "rbac.authorization.k8s.io"]
       resources: ["pods", "componentstatuses", "configmaps", "daemonsets", "deployments",
                   "events", "endpoints", "horizontalpodautoscalers", "ingress", "jobs",
                   "limitranges", "namespaces", "nodes", "persistentvolumes", "persistentvolumeclaims",
                   "resourcequotas", "replicasets", "replicationcontrollers", "serviceaccounts", "services"]
       verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
   ```
   ```bash
   kubectl apply -f role.yaml
   ```

3. **Create a RoleBinding** (`bind.yaml`):
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: app-rolebinding
     namespace: webapps
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: Role
     name: app-role
   subjects:
     - kind: ServiceAccount
       name: jenkins
       namespace: webapps
   ```
   ```bash
   kubectl apply -f bind.yaml
   ```

4. **Create a Secret with a Token** (`secret.yaml`):
   ```yaml
   apiVersion: v1
   kind: Secret
   type: kubernetes.io/service-account-token
   metadata:
     name: mysecretname
     annotations:
       kubernetes.io/service-account.name: jenkins
   ```
   ```bash
   kubectl apply -f secret.yaml -n webapps
   ```
   - Describe the secret to get the token:  
     ```bash
     kubectl describe secret mysecretname -n webapps
     ```
   - Copy this token and store it as a Jenkins credential (e.g., `k8-token`).

### Step 5: Encode Environment Variables and Deploy
1. **Base64 encode environment variables** for Kubernetes Secrets:
   ```bash
   echo -n 'dj0lkx4sa' | base64
   ```
2. **Create a Kubernetes Secret** manifest (e.g., `env-secrets.yaml`):
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: env-secrets
     namespace: webapps
   type: Opaque
   data:
     CLOUDINARY_CLOUD_NAME: ZGo0bGt4NHNh  # base64-encoded
     CLOUDINARY_KEY: ...
     CLOUDINARY_SECRET: ...
     MAPBOX_TOKEN: ...
     DB_URL: ...
     SECRET: ...
   ```
   ```bash
   kubectl apply -f env-secrets.yaml
   ```
3. **Deployment YAML** references these secrets as environment variables.  
4. **Jenkins Pipeline** snippet to deploy:
   ```groovy
   stage('Deploy To EKS') {
     steps {
       withKubeCredentials(kubectlCredentials: [[
         caCertificate: '',
         clusterName: 'EKS-1',
         contextName: '',
         credentialsId: 'k8-token',
         namespace: 'webapps',
         serverUrl: 'https://B7C7C20487B2624AAB0AD54DF1469566.yl4.ap-south-1.eks.amazonaws.com'
       ]]) {
         sh "kubectl apply -f kubernetes-manifests/deployment.yaml"
       }
     }
   }

   stage('Verify Deployment') {
     steps {
       withKubeCredentials(kubectlCredentials: [[
         caCertificate: '',
         clusterName: 'EKS-1',
         contextName: '',
         credentialsId: 'k8-token',
         namespace: 'webapps',
         serverUrl: 'https://B7C7C20487B2624AAB0AD54DF1469566.yl4.ap-south-1.eks.amazonaws.com'
       ]]) {
         sh "kubectl get svc -n webapps"
         sh "kubectl get pods -n webapps"
       }
     }
   }
   ```
5. **Check Logs/Service**:
   - After the pipeline successfully runs, confirm pods and services are up:
     ```bash
     kubectl get pods -n webapps
     kubectl get svc -n webapps
     ```
   - If you exposed your service via a **LoadBalancer**, note the external IP to access your app in the browser.

---

## Key Takeaways
1. **Three-Tier Architecture**: Gained a deeper understanding of structuring an application into presentation, logic, and data layers.  
2. **DevOps Tools Integration**: Hands-on exposure to **Docker**, **Jenkins**, **SonarQube**, **Trivy**, and **Kubernetes**.  
3. **CI/CD Pipeline**: Automated building, testing, security scanning, and deployment.  
4. **Environment Segregation**: Clear separation of local, development, and production environments.  
5. **Security Best Practices**: Managing secrets securely in `.env` files and Kubernetes Secrets.  
6. **Scalability**: Leveraging AWS EKS and Kubernetes for production-ready deployments.

---

## Acknowledgement
A special thank you to **Mr. Aditya Jaiswal** for his invaluable guidance and tutorials on the [DevOps Shack YouTube channel](https://www.youtube.com/@DevOpsShack). His detailed explanations greatly contributed to the success of this project.

---

## Conclusion
This **Three Tier Full Stack Project** walks through a complete application life cycle, showcasing how to configure a CI/CD pipeline with **Jenkins**, containerize applications using **Docker**, analyze code quality with **SonarQube**, scan for vulnerabilities with **Trivy**, and finally deploy to a production-ready **AWS EKS** cluster. By following this guide, you should have a deeper understanding of how to automate, secure, and scale applications in a cloud environment using modern DevOps tools and best practices.

*Thank you for exploring the Three Tier Full Stack Project! If you have suggestions or wish to contribute, please open an issue or PR on the [GitHub repository](https://github.com/ShubhamStunner/3-Tier-Full-Stack.git).*
```

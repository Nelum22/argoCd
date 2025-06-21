# CI/CD Pipeline with Jenkins, SonarQube, Docker, Argo CD, and EKS

This project demonstrates the setup and deployment of a CI/CD pipeline using **Jenkins**, **SonarQube**, and **Argo CD** on AWS infrastructure. The pipeline includes code quality checks, Docker image creation, and deployment of a simple static application to an **EKS cluster**.

---

## Table of Contents

- [Project Overview](#project-overview)  
- [Architecture](#architecture)  
- [Prerequisites](#prerequisites)  
- [Setup Instructions](#setup-instructions)  
  - [Setting up Jenkins](#setting-up-jenkins)  
  - [Setting up SonarQube](#setting-up-sonarqube)  
  - [Setting up EKS Cluster](#setting-up-eks-cluster)  
  - [Setting up Argo CD](#setting-up-argo-cd)  
- [Pipeline Workflow](#pipeline-workflow)  
- [Deployment](#deployment)  
- [Conclusion](#conclusion)  

---

## Project Overview

This project involves setting up a CI/CD pipeline to automate the process of building, testing, and deploying a **Simple static application**. The key components used in this project are:

- **Jenkins** for continuous integration  
- **SonarQube** for code quality analysis  
- **Docker** for containerization  
- **Docker Hub** for image repository  
- **Amazon EKS** (Elastic Kubernetes Service) for container orchestration  
- **Argo CD** for continuous deployment  

---

## Architecture

```
Developer → GitHub → Jenkins → SonarQube
                             ↓
                     Docker Build → Docker Hub
                             ↓
                 Update Kubernetes YAML → GitHub
                             ↓
                          Argo CD → EKS
```

---

## Prerequisites

Before you begin, ensure you have the following:

- AWS account with necessary permissions  
- AWS CLI installed and configured  
- `kubectl` installed  
- Docker installed  
- Jenkins, SonarQube, and Argo CD instances set up on AWS EC2  
- GitHub and Docker Hub accounts  

---

## Setup Instructions

### Setting up Jenkins

**Launch an EC2 Instance:**

- Use an Amazon Linux 2 AMI.  
- Configure security groups to allow HTTP (port 80), HTTPS (port 443), and port 8080 for Jenkins.

**Install Jenkins:**

```bash
sudo yum update -y
sudo yum install java-11-openjdk -y
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
sudo yum install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

**Configure Jenkins:**

- Access Jenkins through its web interface at `http://<EC2_PUBLIC_IP>:8080`  
- Complete the setup wizard.  
- Install required plugins (Git, Docker, SonarQube Scanner).  
- Add credentials for:
  - GitHub
  - DockerHub
  - SonarQube

---

### Setting up SonarQube

**Launch an EC2 Instance:**

- Use an Amazon Linux 2 AMI.  
- Configure security groups to allow HTTP and port 9000.

**Install SonarQube:**

```bash
sudo yum install java-11-openjdk -y
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.1.0.73491.zip
unzip sonarqube-*.zip
cd sonarqube-*/
sudo chown -R ec2-user:ec2-user .
./bin/linux-x86-64/sonar.sh start
```

- Access SonarQube at `http://<EC2_PUBLIC_IP>:9000`

---

### Setting up EKS Cluster

```bash
# Install eksctl if not already installed
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Create EKS cluster
eksctl create cluster --name java-maven-cluster --nodes 2 --region <your-region>

# Configure kubectl
aws eks --region <your-region> update-kubeconfig --name java-maven-cluster
```

---

### Setting up Argo CD

```bash
# Create Argo CD namespace
kubectl create namespace argocd

# Install Argo CD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Port forward Argo CD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get initial password for login
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

---

## Pipeline Workflow

1. **Run Jenkins Pipeline in Docker Container:**  
   - Configure Jenkins to use a Docker-based agent.

2. **Code Quality Check with SonarQube:**  
   - Integrate SonarQube into the Jenkins pipeline to scan the code.

3. **Build Application and Docker Image:**  
   - Use the Dockerfile to package the Maven application into an image.

4. **Push Docker Image to Docker Hub:**  
   - Authenticate with Docker Hub and push the image.

5. **Update Deployment File and Push to GitHub:**  
   - Modify the Kubernetes deployment YAML with the new image tag.  
   - Push the updated file back to the GitHub repo.

6. **Deploy with Argo CD:**  
   - Argo CD detects the change in the GitHub repo and syncs the updated manifest to the EKS cluster.

---

## Deployment

Follow the pipeline steps to ensure the application is built, tested, and deployed seamlessly:

- Trigger the Jenkins pipeline.  
- Monitor code quality analysis results in SonarQube.  
- Check Docker Hub for the new image.  
- Confirm successful deployment in Argo CD.  
- Access the application via the LoadBalancer URL of the EKS service.

---

## Conclusion

This README provides a comprehensive guide to setting up a modern, containerized CI/CD pipeline using Jenkins, SonarQube, and Argo CD on AWS. It ensures code quality, automates Docker image creation, and manages deployment of a Java Maven application to a Kubernetes-based environment (EKS).

---

## References

- https://www.jenkins.io/  
- https://docs.sonarqube.org/  
- https://www.docker.com/  
- https://kubernetes.io/  
- https://argo-cd.readthedocs.io/

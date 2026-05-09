# 🚀 InsureOps - DevOps CI/CD Automation Project

## 📌 Project Overview

**InsureOps** is a complete DevOps CI/CD automation project designed to automate the build, testing, containerization, and deployment process of a Java-based insurance application.

As the application scaled, managing deployments manually became difficult and error-prone. To solve this, a fully automated DevOps pipeline was implemented using industry-standard tools such as Jenkins, Docker, GitHub, SonarQube, and AWS.

The project demonstrates real-world DevOps practices including Continuous Integration (CI), Continuous Deployment (CD), Infrastructure Automation, Containerization, and Cloud Deployment.

---

# 🏗️ Project Architecture

```text
Developer → GitHub → Jenkins Pipeline → Maven Build → SonarQube Scan 
→ Docker Build → DockerHub → AWS EC2 Deployment
```

---

# 🎯 Project Requirements

## 1️⃣ Automated Deployment

Whenever developers push code changes to the `main` branch of the Git repository, Jenkins automatically triggers the deployment pipeline.

---

## 2️⃣ CI/CD Pipeline

The Jenkins pipeline performs the following tasks automatically:

- Pull latest source code from GitHub
- Build the application using Maven
- Run code quality analysis using SonarQube
- Create Docker image
- Push Docker image to DockerHub
- Deploy containerized application on AWS EC2

---

# 🛠️ DevOps Tools Used

| Tool | Purpose |
|------|----------|
| Git | Version control and source code management |
| GitHub | Remote repository hosting |
| Jenkins | CI/CD automation server |
| Maven | Build automation and dependency management |
| Docker | Containerization platform |
| SonarQube | Code quality and security analysis |
| AWS EC2 | Cloud infrastructure hosting |
| DockerHub | Docker image repository |

---

# ☁️ AWS Infrastructure Setup

- Created Ubuntu EC2 instance (`t2.medium`)
- Configured security groups and networking
- Installed Jenkins, Docker, Maven, AWS CLI
- Hosted application deployment environment

---

# 📋 Project Workflow

1. Developer pushes code to GitHub
2. GitHub webhook triggers Jenkins pipeline
3. Jenkins pulls latest source code
4. Maven builds and packages the application
5. SonarQube performs code quality analysis
6. Docker image is created
7. Docker image is pushed to DockerHub
8. Application container is deployed on AWS EC2

---

# ⚙️ Server Setup

## 🔹 Install Jenkins

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre -y

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins -y
```

---

## 🔹 Install Docker

```bash
sudo apt install docker.io -y
sudo systemctl start docker

sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu

newgrp docker

sudo chmod 777 /var/run/docker.sock
```

---

## 🔹 Install SonarQube

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

---

## 🔹 Install Maven

```bash
sudo apt install maven -y
```

---

## 🔹 Install AWS CLI

```bash
sudo apt install unzip -y

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
-o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install
```

---

# 🔌 Jenkins Plugins Used

- Stage View
- Maven Integration
- SonarQube Scanner
- AWS Credentials
- S3 Publisher
- Docker

---

# 🔐 Jenkins Configuration

## Configure Tools
Navigate to:

```text
Manage Jenkins → Tools
```

Configure:
- Maven
- SonarQube Scanner
- JDK

---

## Configure Credentials

Add:
- GitHub Credentials
- DockerHub Credentials
- AWS Credentials
- SonarQube Token

---

# 🔗 GitHub Webhook Setup

Configure GitHub webhook to trigger Jenkins automatically on every push event.

```text
GitHub Repository → Settings → Webhooks
```

Webhook URL:

```text
http://<JENKINS_PUBLIC_IP>:8080/github-webhook/
```

---

# 📦 Jenkins Pipeline

## Jenkinsfile

```groovy
pipeline {
    agent any 

    tools{
        maven 'maven'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        S3_BUCKET = "project-insure-me-build-artifacts-store"
        REGION = "ap-south-1"
        warFile = "target/Insurance-0.0.1-SNAPSHOT.jar"
    }

    stages {

        stage('Code Pull') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/your-username/insureops.git'
                    ]]
                )
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=InsureOps \
                    -Dsonar.projectName=InsureOps \
                    -Dsonar.sources=src \
                    -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false
            }
        }

        stage('Push Artifact to S3') {
            steps {
                withCredentials([
                    aws(
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        credentialsId: 'aws-cred',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                    aws s3 cp ${warFile} \
                    s3://${S3_BUCKET}/Artifacts/ \
                    --region ${REGION}
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t your-dockerhub-username/insureops .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-cred',
                        passwordVariable: 'dockerHubPassword',
                        usernameVariable: 'dockerHubUser'
                    )
                ]) {

                    sh '''
                    docker login -u ${dockerHubUser} -p ${dockerHubPassword}
                    docker push your-dockerhub-username/insureops
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                docker run -itd \
                --name insureops \
                -p 8089:8081 \
                your-dockerhub-username/insureops
                '''
            }
        }
    }
}
```

---

# 🐳 Docker Commands

## Build Docker Image

```bash
docker build -t insureops .
```

## Run Docker Container

```bash
docker run -itd --name insureops -p 8089:8081 insureops
```

---

# 📈 CI/CD Benefits Achieved

✅ Automated Build Process  
✅ Continuous Integration  
✅ Continuous Deployment  
✅ Faster Release Cycle  
✅ Improved Code Quality  
✅ Containerized Deployment  
✅ Reduced Manual Errors  
✅ Scalable Infrastructure  

---

# 📸 Screenshots

Add project screenshots here:

```text
images/
├── jenkins-dashboard.png
├── sonar-dashboard.png
├── docker-container.png
├── pipeline-success.png
└── aws-deployment.png
```

---

# 🚀 Future Enhancements

- Kubernetes Deployment
- Terraform Infrastructure Automation
- Monitoring with Prometheus & Grafana
- Helm Charts
- Blue-Green Deployment
- GitHub Actions Integration

---

# 👨‍💻 Author

### Ketan Dhadve

DevOps Engineer | Cloud Enthusiast | CI/CD Automation

---

# 📚 Conclusion

InsureOps successfully demonstrates a real-world DevOps CI/CD implementation using Jenkins, Docker, SonarQube, GitHub, and AWS.

The project automates the entire software delivery lifecycle, ensuring reliable, scalable, and efficient deployments while reducing manual intervention and improving development productivity.

---

# ⭐ If you like this project

Give this repository a ⭐ on GitHub.

# 🚀 Steps to Run InsureOps DevOps Project

# 📌 Step 1 — Launch AWS EC2 Instance

- Login to AWS Console
- Open EC2 Dashboard
- Launch Ubuntu Server Instance (`t2.medium`)
- Configure Security Group:
  - 22 → SSH
  - 8080 → Jenkins
  - 9000 → SonarQube
  - 8081/8089 → Application
- Connect to EC2 using SSH

```bash
ssh -i key.pem ubuntu@<EC2-PUBLIC-IP>
```

---

# 📌 Step 2 — Update Server

```bash
sudo apt update && sudo apt upgrade -y
```

---

# 📌 Step 3 — Install Java

```bash
sudo apt install fontconfig openjdk-21-jre -y
```

Verify Java:

```bash
java -version
```

---

# 📌 Step 4 — Install Jenkins

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update

sudo apt install jenkins -y
```

Start Jenkins:

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

Check status:

```bash
sudo systemctl status jenkins
```

---

# 📌 Step 5 — Access Jenkins

Open browser:

```text
http://<EC2-PUBLIC-IP>:8080
```

Get Jenkins admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

- Install Suggested Plugins
- Create Admin User

---

# 📌 Step 6 — Install Docker

```bash
sudo apt install docker.io -y
```

Start Docker:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Add users to Docker group:

```bash
sudo usermod -aG docker ubuntu
sudo usermod -aG docker jenkins
```

Apply changes:

```bash
newgrp docker
```

Set Docker permissions:

```bash
sudo chmod 777 /var/run/docker.sock
```

Verify Docker:

```bash
docker --version
```

---

# 📌 Step 7 — Install Maven

```bash
sudo apt install maven -y
```

Verify Maven:

```bash
mvn -version
```

---

# 📌 Step 8 — Install SonarQube

Run SonarQube container:

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

Access SonarQube:

```text
http://<EC2-PUBLIC-IP>:9000
```

Default Login:

```text
Username: admin
Password: admin
```

Generate SonarQube Token.

---

# 📌 Step 9 — Install AWS CLI

```bash
sudo apt install unzip -y

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
-o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install
```

Verify AWS CLI:

```bash
aws --version
```

---

# 📌 Step 10 — Install Jenkins Plugins

Go to:

```text
Manage Jenkins → Plugins
```

Install:

- Docker
- Stage View
- Maven Integration
- SonarQube Scanner
- AWS Credentials
- Pipeline
- GitHub Integration

Restart Jenkins after installation.

---

# 📌 Step 11 — Configure Jenkins Tools

Navigate to:

```text
Manage Jenkins → Tools
```

Configure:
- JDK
- Maven
- SonarQube Scanner

---

# 📌 Step 12 — Add Jenkins Credentials

Go to:

```text
Manage Jenkins → Credentials
```

Add:
- GitHub Credentials
- DockerHub Credentials
- AWS Credentials
- SonarQube Token

---

# 📌 Step 13 — Create GitHub Repository

Create repository on GitHub:

Example:

```text
insureops
```

Push project code:

```bash
git init
git add .
git commit -m "Initial Commit"

git remote add origin https://github.com/USERNAME/insureops.git

git branch -M main

git push -u origin main
```

---

# 📌 Step 14 — Configure GitHub Webhook

GitHub Repository:

```text
Settings → Webhooks → Add Webhook
```

Payload URL:

```text
http://<EC2-PUBLIC-IP>:8080/github-webhook/
```

Content Type:

```text
application/json
```

Select:

```text
Just the push event
```

Save webhook.

---

# 📌 Step 15 — Create Jenkins Pipeline Project

Dashboard → New Item

Select:
```text
Pipeline Project
```

Enable:
```text
GitHub hook trigger for GITScm polling
```

---

# 📌 Step 16 — Add Jenkinsfile

Create file:

```text
Jenkinsfile
```

Add your pipeline script.

Commit and push:

```bash
git add .
git commit -m "Added Jenkins Pipeline"
git push
```

---

# 📌 Step 17 — Run Pipeline

Jenkins automatically triggers pipeline after push.

Pipeline Stages:
- Code Pull
- Build
- Test
- SonarQube Scan
- Docker Build
- Docker Push
- Deployment

---

# 📌 Step 18 — Verify Deployment

Check running containers:

```bash
docker ps
```

Access application:

```text
http://<EC2-PUBLIC-IP>:8089
```

---

# 📌 Step 19 — Verify Docker Image

```bash
docker images
```

---

# 📌 Step 20 — Verify SonarQube Report

Open:

```text
http://<EC2-PUBLIC-IP>:9000
```

Check:
- Code Smells
- Bugs
- Vulnerabilities
- Quality Gate

---

# 🎯 Final Outcome

✅ Automated CI/CD Pipeline  
✅ Continuous Integration  
✅ Continuous Deployment  
✅ Dockerized Application  
✅ AWS Cloud Deployment  
✅ SonarQube Code Analysis  
✅ Production-Ready DevOps Workflow  

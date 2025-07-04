# 🚀 Reddit Clone Deployment Using Jenkins CI/CD, DevSecOps & EKS

This project demonstrates how to build and deploy a **Reddit Clone App** using a complete **CI/CD pipeline** with Jenkins, Docker, SonarQube, Trivy, Dependency-Check, and deploy it to AWS EKS with Kubernetes and Ingress.

---

## 📦 Tech Stack

- Jenkins (CI/CD)
- Docker (Containerization)
- SonarQube (Static Code Analysis)
- Trivy (Vulnerability Scanning)
- OWASP Dependency Check (Security)
- Kubernetes + EKS (Deployment)
- Ingress (Routing)
- Node.js + React (Frontend)
- AWS CLI & eksctl (Cloud Infra)

---

## 🧰 Prerequisites

- AWS EC2 Ubuntu 22.04 instance
- Port access in Security Groups:
  - `8080` (Jenkins)
  - `9000` (SonarQube)
  - `3000` (App Preview)
- IAM user with EKS permissions
- Domain or Host entry for Ingress (optional)

---

## 🏗️ Setup Instructions

### 1️⃣ Install Jenkins

```bash
#!/bin/bash
sudo apt update -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins
```
Get Jenkins password:

```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

## 2️⃣ Install Docker & SonarQube


```
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER
newgrp docker
sudo chmod 777 /var/run/docker.sock

docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

## 3️⃣ Install Trivy (File & Image Scanner)

```sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```


### 4️⃣ Install Jenkins Plugins

To enable all necessary integrations, install the following plugins in Jenkins:

#### 📍 Steps:
1. Open Jenkins in your browser: `http://<EC2-IP>:8080`
2. Go to: **Manage Jenkins → Plugin Manager → Available**
3. Search for and install the following plugins:

Eclipse Temurin Installer

SonarQube Scanner

NodeJS

OWASP Dependency Check

Docker

Kubernetes

Pipeline: Stage View



### 5️⃣ Global Tool Configuration

Configure Jenkins global tools to enable pipeline steps such as compilation, scanning, and building.

#### 📍 Steps:
1. Go to: **Manage Jenkins → Global Tool Configuration**
2. Configure the following tools:

---

#### 🧩 JDK

- **Name**: `jdk17`
- ✅ Check: **Install automatically**
- **Version**: `jdk-17.0.8.1+1` (from Temurin)

---

#### 🟢 Node.js

- **Name**: `node16`
- ✅ Check: **Install automatically**
- **Version**: `16.2.0`

---

#### 🔍 Sonar Scanner

- **Name**: `sonar-scanner`
- ✅ Check: **Install automatically**
- **Version**: `Latest available`

---

#### 🛡️ OWASP Dependency Check

- **Name**: `DP-Check`
- ✅ Check: **Install automatically**
- **Version**: `Latest available`

---

✅ After saving the configuration, these tools will be available for your pipeline stages.

> 💡 These tool names (e.g., `jdk17`, `node16`, etc.) should **match exactly** when referenced in your `Jenkinsfile`.

### 6️⃣ Configure SonarQube in Jenkins

Integrate SonarQube with Jenkins to perform **static code analysis** during the CI pipeline.

---

#### 🛠️ 1. Generate SonarQube Token

1. Open SonarQube: `http://<EC2_PUBLIC_IP>:9000`
2. Navigate to: **Administration → Security → Tokens**
3. Generate a new token (e.g., name it `jenkins-token`)
4. Copy the token (it will only be shown once)

---

#### 🔐 2. Add Token to Jenkins Credentials

1. Go to: **Manage Jenkins → Credentials → (Global) → Add Credentials**
2. Fill in the following:

| Field        | Value           |
|--------------|------------------|
| **Kind**     | Secret text       |
| **Secret**   | `<paste generated token>` |
| **ID**       | `Sonar-token`     |
| **Description** | `SonarQube access token` (optional) |

---

#### ⚙️ 3. Configure SonarQube Server in Jenkins

1. Go to: **Manage Jenkins → Configure System**
2. Scroll to **SonarQube Servers** section
3. Click **Add SonarQube**
4. Fill in:

| Field              | Value                            |
|--------------------|----------------------------------|
| **Name**           | `sonar-server`                   |
| **Server URL**     | `http://<EC2_PUBLIC_IP>:9000`    |
| **Authentication Token** | `Sonar-token`             |

✅ Check the box **Enable injection of SonarQube server configuration** if available.

---

#### 🔁 4. Add Webhook in SonarQube

1. In SonarQube: **Administration → Configuration → Webhooks**
2. Click **Create**
3. Set:

| Field     | Value                                 |
|-----------|----------------------------------------|
| **Name**  | `jenkins-webhook`                      |
| **URL**   | `http://<EC2_PUBLIC_IP>:8080/sonarqube-webhook/` |

📌 This enables SonarQube to notify Jenkins once analysis is complete.

---

✅ Your SonarQube is now fully integrated with Jenkins and ready to run scans during CI/CD pipeline.

```
# AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws configure

# kubectl
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin

# eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /bin
```
8️⃣ Create EKS Cluster

```
eksctl create cluster --name reddit-clone --region eu-north-1 --node-type t3.medium --nodes 3
kubectl get nodes
```

### 9️⃣ Add Kubernetes Credentials in Jenkins

To allow Jenkins to deploy applications to your EKS cluster, add the Kubernetes config file as a credential.

---

#### 📁 Step-by-Step:

1. In your terminal (on the Jenkins host or where `eksctl` was used), locate the kubeconfig file:
   ```bash
   cat ~/.kube/config

## 🔁 Jenkins Pipeline (Example)

```
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        AWS_REGION = 'eu-north-1'
        CLUSTER_NAME = 'reddit-clone'
    }
    stages {
        stage('Clean Workspace') {
            steps { cleanWs() }
        }
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/helloworld064/reddit-clone-k8s.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Reddit \
                        -Dsonar.projectKey=Reddit'''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps { sh 'npm install' }
        }
        stage('OWASP FS Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }
        stage('Docker Stop & Remove Old Container') {
            steps {
                sh '''
                    docker stop reddit || true
                    docker rm reddit || true
                '''
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker build -t reddit .'
                        sh 'docker tag reddit rohitjain064/reddit:latest'
                        sh 'docker push rohitjain064/reddit:latest'
                    }
                }
            }
        }
        stage('Run Container Locally') {
            steps {
                sh 'docker run -d --name reddit -p 3000:3000 rohitjain064/reddit:latest'
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image rohitjain064/reddit:latest > trivy.txt'
            }
        }
        stage('Configure AWS & Connect to EKS') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'aws-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )]) {
                        withEnv([
                            "AWS_ACCESS_KEY_ID=${env.AWS_ACCESS_KEY_ID}",
                            "AWS_SECRET_ACCESS_KEY=${env.AWS_SECRET_ACCESS_KEY}",
                            "AWS_REGION=${env.AWS_REGION}"
                        ]) {
                            sh '''
                                aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME
                                kubectl get nodes
                            '''
                        }
                    }
                }
            }
        }
        stage("Deploy to EKS") {
            steps {
                sh '''
                    kubectl apply -f deployment.yml
                '''
            }
        }
    }
}
```


![reddit-with-jenkins](https://github.com/user-attachments/assets/551b34e1-ac9f-4685-b5ca-fa823f1a6d62)


![reddit-with-sonarqube](https://github.com/user-attachments/assets/6a6045fc-f5a1-4fd4-9066-0901bc1ad7a8)

![reddit-with-ssl](https://github.com/user-attachments/assets/c46bd97d-ab3e-49ec-b0c5-d77270b0618f)



----
## Cleanup
1--Delete EKS Cluster
```eksctl delete cluster --region=eu-south-1 --name=reddit-clone```

2--Delete EC2 Instance with below Terraform Command
```terraform destroy```



## 📚 References

- [Jenkins Docs](https://www.jenkins.io/doc/)
- [SonarQube](https://www.sonarqube.org/)
- [Trivy](https://github.com/aquasecurity/trivy)
- [DockerHub](https://hub.docker.com/)
- [AWS EKS](https://docs.aws.amazon.com/eks/)
- [eksctl](https://eksctl.io/)







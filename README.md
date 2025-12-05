# 🚀 Node.js CI/CD Pipeline using Jenkins, Docker, and AWS EC2

This project demonstrates a complete CI/CD pipeline for automating the build, test, containerization, and deployment of a Node.js application using Jenkins Declarative Pipeline, Docker, and AWS EC2.

---

## 📌 Project Architecture

GitHub → Jenkins → Docker Build → Docker Hub → AWS EC2 (Deployment)

---

## 🛠️ Technologies Used

- **Node.js & npm**
- **Jenkins (Declarative Pipeline)**
- **Docker**
- **Docker Hub**
- **AWS EC2 (Ubuntu Server)**
- **SSH Agent Plugin**
- **GitHub PAT**
- **Shell Scripting**

---

## 🔧 CI/CD Workflow Steps

### **1. GitHub → Jenkins**
- Code is stored in a GitHub repository.
- Jenkins pulls the repository using **GitHub PAT credentials**.

### **2. Install Dependencies**
Jenkins runs:
bash
npm install
Run Tests
npm test
Build Docker Image

Jenkins builds a Docker image using the project's Dockerfile:

docker build -t himanshuchaudhari02/node-ci-cd-app .

 Push Image to Docker Hub

Authenticated push using Jenkins credentials:

docker push himanshuchaudhari02/node-ci-cd-app

 Deploy on AWS EC2

Using SSH Agent Plugin, Jenkins remotely connects to EC2 and executes:

docker pull himanshuchaudhari02/node-ci-cd-app
docker stop nodeapp || true
docker rm nodeapp || true
docker run -d --name nodeapp -p 80:3000 himanshuchaudhari02/node-ci-cd-app


EC2 runs the latest version of the containerized Node.js app.

🧩 Jenkinsfile (Pipeline as Code)
pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = credentials('dockerhub')
        DOCKER_IMAGE = "himanshuchaudhari02/node-ci-cd-app"
        DEPLOY_SERVER = "ubuntu@13.204.238.92"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/Himachaudhari/node-ci-cd-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE ."
            }
        }

        stage('Login to Docker Hub') {
            steps {
                sh "echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh "docker push $DOCKER_IMAGE"
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no $DEPLOY_SERVER '
                        docker pull $DOCKER_IMAGE &&
                        docker stop nodeapp || true &&
                        docker rm nodeapp || true &&
                        docker run -d --name nodeapp -p 80:3000 $DOCKER_IMAGE
                    '
                    """
                }
            }
        }
    }
}
📷 Output Screenshot (CI/CD Successful)

✔ Build Successful
✔ Docker Image Pushed
✔ EC2 Deployment Completed
✔ Website accessible via EC2 Public IP

⭐ Features

Fully automated CI/CD pipeline

Zero manual deployment

Secure credential management

Docker-based portability

Production-grade EC2 deployment

📬 Contact

For questions or collaboration:

Himanshu Chaudhari
👨‍💻 LinkedIn | 🌐 GitHub | 📧 Email

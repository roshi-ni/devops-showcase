# CI/CD Pipeline with Jenkins, SonarQube, and Docker

This project demonstrates a complete CI/CD pipeline for the Spring PetClinic application using Jenkins.  
The pipeline includes Git integration, Maven build, SonarQube code analysis, Docker image build & push, and container deployment.

---

## Objective
- Automate the build, test, and deployment of the Spring PetClinic application.  
- Integrate code quality analysis with SonarQube.  
- Build and push Docker images to Docker Hub.  
- Deploy the application container automatically after each build.

---

## Table of Contents
1. [Project Overview](#project-overview)  
2. [Prerequisites](#prerequisites)  
3. [Environment Setup](#environment-setup)  
4. [Jenkins Pipeline Code](#jenkins-pipeline-code)  
5. [Stage-by-Stage Walkthrough](#stage-by-stage-walkthrough)  
6. [Build & Run Instructions](#build--run-instructions)  
7. [Troubleshooting](#troubleshooting)  
8. [Conclusion](#conclusion)  

---

## Project Overview
The CI/CD pipeline automates the workflow for the Spring PetClinic application.  
Each commit to the repository triggers:
1. Code checkout from GitHub  
2. Build and test with Maven  
3. Code quality scanning via SonarQube  
4. Docker image build and push to Docker Hub  
5. Deployment of the latest container on the target server  

This ensures continuous integration and continuous delivery with minimal manual intervention.

---

## Prerequisites

**Server Requirements:**  
- Linux t2.medium  
- Jenkins installed and running (Java 21)  
- Docker installed and configured (on agent)  
- Java 17 and Maven installed (on agent node)  

**Accounts / Access:**  
- GitHub account with repository access  
- Docker Hub account (credentials stored in Jenkins: credentialsId `docker`)  
- SonarQube server running (with token configured)  

**Ports Used:**  
- Jenkins: 8080  
- SonarQube: 9000  
- Application: 9090  

---

## Environment Setup

1. **Install Java & Maven** on agent node  
2. **Install Docker & Docker Compose** on agent node  
3. **Add Jenkins user to Docker group**  
```bash
sudo usermod -aG docker jenkins
sudo systemctl restart docker
Configure Jenkins:

Install required plugins: Pipeline, Git, Docker, SonarQube Scanner

Add SonarQube server under Manage Jenkins → Configure System

Add Docker Hub and SonarQube credentials in Manage Jenkins → Credentials

Add agent node credentials (ec2-user)

Jenkins Pipeline Code
groovy
Copy code
pipeline {
    agent { label 'dev' }

    stages {
        stage('git') {
            steps {
                git branch: 'main', url: 'https://github.com/roshi-ni/spring-petclinic.git'
            }
        }
        stage('maven') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('sonar') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh ''' 
                    mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=sonar-project \
                    -Dsonar.projectName='sonar-project' \
                    -Dsonar.host.url=http://13.127.96.179:9000 \
                    -Dsonar.token=sqp_efab546cd59656f37e809f9a8749776881710910
                    '''
                }
            }
        }
        stage('docker build & push') {
            steps {
                withDockerRegistry(credentialsId: 'docker', url: 'https://index.docker.io/v1/') {
                   sh 'docker build -t roshini10/petclinic:${BUILD_NUMBER} -f Dockerfile .'
                   sh 'docker push roshini10/petclinic:${BUILD_NUMBER}'
                }
            }
        }
        stage('deploy container'){
            steps{
                sh 'docker rm -f petclinic || true'
                sh 'docker pull roshini10/petclinic:${BUILD_NUMBER}'
                sh 'docker run -d --name petclinic -p 9090:8080 roshini10/petclinic:${BUILD_NUMBER}'
            }
        }
    }
}
Stage-by-Stage Walkthrough
* Git Checkout

Pulls the latest code from the main branch of GitHub

* Maven Build

Runs mvn clean install to build the project and run tests

* SonarQube Analysis

Executes mvn sonar:sonar with project key, name, and authentication token

Sends results to SonarQube server at http://13.127.96.179:9000

First create the project in SonarQube dashboard; generate a token for Jenkins

* Docker Build & Push

Builds Docker image tagged with ${BUILD_NUMBER}

Pushes image to Docker Hub: roshini10/petclinic

* Deploy Container

Removes any existing petclinic container

Pulls the new image from Docker Hub

Runs the container on port 9090

* Build & Run Instructions
Trigger a Jenkins build manually or via Git webhook

Monitor build logs in Jenkins console

After success, check running container:

docker ps -a
Access application at:

http://<server-public-ip>:9090
Troubleshooting
SonarQube connection error:

Verify SonarQube server is running on port 9000

Check token validity

Docker permission denied:

Ensure jenkins user is in Docker group

Port 9090 already in use:

Stop or reassign existing service on port 9090

Conclusion
This pipeline provides a complete CI/CD workflow for Spring PetClinic.
Each commit is automatically built, analyzed, containerized, and deployed.
It reduces manual work, ensures consistent builds, and provides quick feedback through SonarQube.

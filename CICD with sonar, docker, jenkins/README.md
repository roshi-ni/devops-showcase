# CI/CD Using SonarQube and Docker 
Project Name: Spring PetClinic — CI/CD Pipeline with Jenkins, SonarQube, and Docker
Short Description:
This project demonstrates a complete CI/CD pipeline for the Spring PetClinic application using Jenkins. The pipeline includes Git integration, Maven build, SonarQube code analysis, Docker image build & push, and container deployment.
________________________________________
## Objective
•	Automate the build, test, and deployment of the Spring PetClinic application.
•	Integrate code quality analysis with SonarQube.
•	Build and push Docker images to Docker Hub.
•	Deploy the application container automatically after each build.
________________________________________
## Table of Contents
1.	Project Overview
2.	Prerequisites
3.	Environment Setup
4.	Jenkins Pipeline Code
5.	Stage-by-Stage Walkthrough
6.	Build & Run Instructions
7.	Troubleshooting
8.	Conclusion
________________________________________
## 1. Project Overview
The CI/CD pipeline automates the workflow for the Spring PetClinic application. Each commit to the repository triggers:
1.	Code checkout from GitHub.
2.	Build and test with Maven.
3.	Code quality scanning via SonarQube.
4.	Docker image build and push to Docker Hub.
5.	Deployment of the latest container on the target server.
This ensures continuous integration and continuous delivery with minimal manual intervention.
________________________________________
## 2. Prerequisites
Server Requirements:
•	Linux t2.medium
•	Jenkins installed and running (java 21)
•	Docker installed and configured(in agent)
•	Java 17 and Maven installed(in agent node)
Accounts/Access:
•	GitHub account with repository access
•	Docker Hub account with credentials stored in Jenkins (credentialsId: docker)
•	SonarQube server running (with token configured)
Ports Used:
•	Jenkins: 8080
•	SonarQube: 9000
•	Application: 9090
________________________________________
## 3. Environment Setup
1.	Install Java & Maven in agent node
2.	Install Docker in agent node
3.	Install docker compose .
4.	Adding jenkins user is in docker group.
sudo usermod -aG docker jenkins
sudo systemctl restart docker

3.	Configure Jenkins :
o	Install required plugins: Pipeline, Git, Docker, SonarQube Scanner.
o	Add SonarQube server under Manage Jenkins → Configure System.
o	Add Docker Hub and sonarQube credentials in Manage Jenkins → Credentials.
o	Add ec2-user credentials for node configuration.
________________________________________
## 4. Jenkins Pipeline Code
pipeline {
    agent {
        label 'dev'
    }

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
________________________________________
## 4.	Stage-by-Stage Walkthrough
Node configuration:
Add a agent node -->

1.	Git Checkout
o	Pulls the latest code from the main branch of GitHub.

2.	Maven Build
o	Runs mvn clean install to build the project and run tests.


3.	SonarQube Analysis
o	Executes mvn sonar:sonar with project key, name, and authentication token.
o	Sends results to SonarQube server at http://13.127.96.179:9000.
o	First create project in sonarqube dashboard , after creating token it will give you the code .












4.	Docker Build & Push
o	Builds Docker image tagged with ${BUILD_NUMBER}.
o	Pushes image to Docker Hub repository: roshini10/petclinic.






5.	Deploy Container
o	Removes any existing petclinic container.
o	Pulls the new image from Docker Hub.
o	Runs the container on port 9090.

________________________________________
6. Build & Run Instructions
1.	Trigger a Jenkins build manually or via Git webhook.
2.	Monitor build logs in Jenkins console.
3.	After success, check running container: docker ps -a
4.	Access application at:
http://<server-public-ip>:9090

________________________________________
7. Troubleshooting
•	SonarQube connection error:
o	Verify SonarQube server is running on port 9000.
o	Check token validity.
•	Docker permission denied:
o	Ensure jenkins user is in docker group.
•	Port 9090 already in use:
o	Stop or reassign existing service on port 9090.
________________________________________
8. Conclusion
This pipeline provides a complete CI/CD workflow for Spring PetClinic. Each commit is automatically built, analyzed, containerized, and deployed. It reduces manual work, ensures consistent builds, and provides quick feedback through SonarQube.




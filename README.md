# BoardgameListingWebApp

## 📖 Description

**Board Game Database Full-Stack Web Application.**  
This web application displays lists of board games and their reviews. While anyone can view the board game lists and reviews, they are required to log in to add/edit the board games and their reviews.  
The **users** can add board games and write reviews, while the **managers** can also edit or delete reviews in addition to the users' permissions.  

## 🛠 Technologies

- Java
- Spring Boot
- Thymeleaf + Thymeleaf Fragments
- HTML5, CSS, JavaScript, Bootstrap
- Spring MVC, Spring Security
- JDBC + H2 Database Engine (In-memory)
- JUnit Test Framework
- Maven
- **Docker** (for building, running Jenkins, and containerizing the application)
- **Docker Hub** (image registry)
- **Minikube** (local Kubernetes cluster)
- **SonarQube** (code quality analysis)
- **Jenkins** (CI/CD pipeline)

## ✨ Features

- Full-Stack Application
- UI components created with Thymeleaf and styled with Twitter Bootstrap
- Authentication and authorization using Spring Security
  - Authentication with username/password
  - Role-based authorization (non-members, users, managers)
- Different roles:
  - **Non-members:** Can only view boardgame lists and reviews
  - **Users:** Can add board games and write reviews
  - **Managers:** Can edit/delete reviews
- Containerized with Docker and deployed on **Minikube**
- Automated CI/CD pipeline with Jenkins + Docker
- Vulnerability scanning with **Trivy**
- Code quality analysis with **SonarQube**
- Rolling updates on Kubernetes using `kubectl set image`
- JUnit test framework for unit testing
- Schema.sql file for initial database setup

## 🚀 How to Run Locally

1. Clone the repository  
2. Open the project in your IDE of choice  
3. Run the Spring Boot application  
4. Use the following default credentials (or sign up as a new user):  
   - `username: bugs | password: bunny` (user role)  
   - `username: daffy | password: duck` (manager role)

---

## ⚙️ CI/CD Setup

The project uses **Jenkins** and **SonarQube** running as Docker containers on the same host.

### 1️⃣ Build and Run Jenkins (Custom Image)

Custom Jenkins image with Maven, Docker CLI, kubectl, and Azure CLI preinstalled:

**Dockerfile:**
```dockerfile
FROM jenkins/jenkins:lts
USER root

RUN apt-get update && apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release \
    unzip \
    jq \
    && rm -rf /var/lib/apt/lists/*

# Install Maven 3.6.3
RUN curl -o /tmp/apache-maven-3.6.3-bin.tar.gz https://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz && \
    tar -xzvf /tmp/apache-maven-3.6.3-bin.tar.gz -C /opt && \
    ln -s /opt/apache-maven-3.6.3 /opt/maven && \
    rm /tmp/apache-maven-3.6.3-bin.tar.gz
ENV MAVEN_HOME=/opt/maven
ENV PATH=$MAVEN_HOME/bin:$PATH

# Install Docker CLI
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg && \
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list && \
    apt-get update && apt-get install -y docker-ce-cli && \
    rm -rf /var/lib/apt/lists/*

# Install Azure CLI
RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash

# Install kubectl
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && \
    rm kubectl

ARG DOCKER_GID=999
RUN groupadd -for -g ${DOCKER_GID} docker && usermod -aG docker jenkins
USER jenkins
WORKDIR /var/jenkins_home
EXPOSE 8080 50000

Run Jenkins container:

docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  custom-jenkins:latest


This allows Jenkins to:

Build Docker images

Push to Docker Hub

Use kubectl to deploy to Minikube

### 2️⃣ Run SonarQube

Create a network and start SonarQube:

docker network create ws-net

docker run -d --name sonarqube \
  --network ws-net \
  -p 9000:9000 \
  -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
  sonarqube:lts

### 3️⃣ Jenkins Pipeline Highlights

The Jenkins pipeline (Jenkinsfile) performs:

Checkout Code → from GitHub

Build & Test → Maven build

SonarQube Analysis → code quality check

Docker Build → builds image tagged with Jenkins BUILD_NUMBER

Trivy Scan → generates vulnerability report and emails CSV

Push to Docker Hub → pushes image for deployment

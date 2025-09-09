pipeline {
    agent any
    tools {
        maven 'Maven-3.9.9'
    }

    environment {
        DOCKERHUB_USERNAME = "shivkumarsingh990"
        DOCKERHUB_REPO     = "boardgame"

    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                userRemoteConfigs: [[url: 'https://github.com/shivkumarsinghbaghel/Boardgame.git',
                credentialsId: 'Github-PAT-TOKEN']]])
            }
        }

        stage('Code Analysis') {
            environment {
                scannerHome = tool 'sonarqube'  
            }
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_TOKEN')]) {
                            sh """
                                mvn clean install sonar:sonar \
                                  -DskipTests \
                                  -Dsonar.projectKey=boardgame-app \
                                  -Dsonar.projectName="Boardgame Project" \
                                  -Dsonar.projectVersion=${BUILD_NUMBER} \
                                  -Dsonar.sources=src/main/java \
                                  -Dsonar.login=$SONAR_TOKEN
                            """
                        }
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn -v'
                sh 'mvn clean package -Dmaven.compiler.plugin.version=3.13.0'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                  docker build -t $DOCKERHUB_USERNAME/$DOCKERHUB_REPO:$BUILD_NUMBER .
                """
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                   trivy image --format json --output trivy-report-$BUILD_NUMBER.json $DOCKERHUB_USERNAME/$DOCKERHUB_REPO:$BUILD_NUMBER
                '''
            }
        }

        stage('Convert report to CSV') {
            steps {
                sh '''
                   jq -r '.Results[]?.Vulnerabilities[]? | [
                        .VulnerabilityID,
                        .PkgName,
                        .InstalledVersion,
                        .FixedVersion,
                        .Severity,
                        .Title
                    ] | @csv' trivy-report-$BUILD_NUMBER.json > trivy-report-$BUILD_NUMBER.csv
                '''
            }
            post {
                success {
                    emailext (
                        subject: "Trivy Scan Report - Build #${BUILD_NUMBER}",
                        body: "Attached is the Trivy vulnerability scan report for build #${BUILD_NUMBER}.",
                        to: "shivkumarsingh990@gmail.com",
                        attachmentsPattern: "trivy-report-${BUILD_NUMBER}.csv"
                    )
                }
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh 'docker push $DOCKERHUB_USERNAME/$DOCKERHUB_REPO:$BUILD_NUMBER'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {

                    sh 'kubectl apply -f deployment-service.yaml'
                    // Update the image to the latest pushed build
                    sh "kubectl set image deployment/boardgame-deployment boardgame=$DOCKERHUB_USERNAME/$DOCKERHUB_REPO:$BUILD_NUMBER"        
                }
            }
        }
    }

    post {
        success {
            emailext(
                to: 'shivkumarsingh990@gmail.com',
                subject: "✅ Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' Succeeded",
                body: "Good news! The job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' completed successfully.\n\nCheck console: ${env.BUILD_URL}"
            )
        }
        failure {
            emailext(
                to: 'shivkumarsingh990@gmail.com',
                subject: "❌ Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' Failed",
                body: "The job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' has failed.\n\nCheck console: ${env.BUILD_URL}"
            )
        }
    }
}

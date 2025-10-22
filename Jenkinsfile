pipeline {
    agent any
    
    environment {
        IMAGE_NAME = "balaakashreddyy/student-management:latest"
        DOCKER_CRED = "dockerhub-credentials"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: "main", url: "https://github.com/AkashReddy123/student-management-portal.git"
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }
        
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CRED}", usernameVariable: "DOCKER_USER", passwordVariable: "DOCKER_PASS")]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${IMAGE_NAME}"
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f k8s/mongo-deployment.yaml"
                sh "kubectl apply -f k8s/student-app-deployment.yaml"
            }
        }
    }
}
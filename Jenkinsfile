pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'dockerhub-credentials' // Jenkins credentials ID for Docker Hub
        GITHUB_CREDENTIALS = 'github-pat'                // Jenkins credentials ID for GitHub PAT
        IMAGE_NAME = 'balaakashreddyy/student-management-portal' // Docker image name
        KUBE_CONFIG_CREDENTIALS = 'kubeconfig'          // Jenkins credentials ID for kubeconfig
        K8S_NAMESPACE = 'default'                        // Kubernetes namespace
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/AkashReddy123/student-management-portal.git',
                    credentialsId: "${GITHUB_CREDENTIALS}"
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:latest -t ${IMAGE_NAME}:${env.BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", 
                                                      usernameVariable: 'DOCKER_USER', 
                                                      passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker push ${IMAGE_NAME}:latest"
                        sh "docker push ${IMAGE_NAME}:${env.BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "${KUBE_CONFIG_CREDENTIALS}", variable: 'KUBECONFIG')]) {
                    sh "kubectl apply -f k8s/ -n ${K8S_NAMESPACE}"
                    sh "kubectl set image deployment/student-app student-app=${IMAGE_NAME}:${env.BUILD_NUMBER} -n ${K8S_NAMESPACE}"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! 🎉"
        }
        failure {
            echo "Pipeline failed! ❌"
        }
    }
}

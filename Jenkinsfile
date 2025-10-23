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
                // Using robust checkout syntax to avoid "not in a git directory" error
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/main']],
                          doGenerateSubmoduleConfigurations: false,
                          extensions: [[$class: 'CleanBeforeCheckout']],
                          userRemoteConfigs: [[
                              url: 'https://github.com/AkashReddy123/student-management-portal.git',
                              credentialsId: "${GITHUB_CREDENTIALS}"
                          ]]
                ])
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
                    sh "docker build -t ${IMAGE_NAME}:latest ."
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
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "${KUBE_CONFIG_CREDENTIALS}", variable: 'KUBECONFIG')]) {
                    sh "kubectl apply -f k8s/ -n ${K8S_NAMESPACE}"
                    sh "kubectl set image deployment/student-app student-app=${IMAGE_NAME}:latest -n ${K8S_NAMESPACE}"
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

pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'dockerhub-credentials'  // Jenkins credentials ID for Docker Hub
        DOCKER_IMAGE = 'balaakashreddyy/student-management-portal'
        KUBE_NAMESPACE = 'student-app'
        KUBE_DEPLOYMENT = 'student-app'
        GIT_REPO = 'https://github.com/AkashReddy123/student-management-portal.git'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'echo "Add your test scripts here if any"'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:latest ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_HUB_CREDENTIALS}") {
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    kubectl set image deployment/${KUBE_DEPLOYMENT} ${KUBE_DEPLOYMENT}=${DOCKER_IMAGE}:latest -n ${KUBE_NAMESPACE} || kubectl apply -f k8s-deployment.yaml -n ${KUBE_NAMESPACE}
                    kubectl rollout status deployment/${KUBE_DEPLOYMENT} -n ${KUBE_NAMESPACE}
                    """
                }
            }
        }

    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}

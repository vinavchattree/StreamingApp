pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '691317217805'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('AWS ECR Login') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} |
                    docker login --username AWS --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    docker build \
                      -t ${ECR_REGISTRY}/streaming-auth:${IMAGE_TAG} \
                      ./backend/authService

                    docker build \
                      -t ${ECR_REGISTRY}/streaming-service:${IMAGE_TAG} \
                      ./backend/streamingService

                    docker build \
                      -t ${ECR_REGISTRY}/streaming-admin:${IMAGE_TAG} \
                      ./backend/adminService

                    docker build \
                      -t ${ECR_REGISTRY}/streaming-chat:${IMAGE_TAG} \
                      ./backend/chatService

                    docker build \
                      --build-arg REACT_APP_AUTH_API_URL=/api \
                      --build-arg REACT_APP_STREAMING_API_URL=/api \
                      --build-arg REACT_APP_STREAMING_PUBLIC_URL=/ \
                      --build-arg REACT_APP_ADMIN_API_URL=/api/admin \
                      --build-arg REACT_APP_CHAT_API_URL=/api/chat \
                      --build-arg REACT_APP_CHAT_SOCKET_URL=/ \
                      -t ${ECR_REGISTRY}/streaming-frontend:${IMAGE_TAG} \
                      ./frontend
                '''
            }
        }

        stage('Push Images') {
            steps {
                sh '''
                    docker push ${ECR_REGISTRY}/streaming-auth:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/streaming-service:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/streaming-admin:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/streaming-chat:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/streaming-frontend:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        success {
            echo 'All five images successfully built and pushed to ECR.'
        }

        failure {
            echo 'Jenkins pipeline failed.'
        }
    }
}

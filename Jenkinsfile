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
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-vinav']
                ]) {
                    sh '''
                        set -e

                        echo "Logging in to Amazon ECR..."

                        aws sts get-caller-identity

                        aws ecr get-login-password \
                          --region ${AWS_REGION} |
                        docker login \
                          --username AWS \
                          --password-stdin ${ECR_REGISTRY}

                        echo "ECR login successful."
                    '''
                }
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "Building authentication service"
                    echo "========================================"

                    docker build \
                      -t ${ECR_REGISTRY}/streaming-auth:${IMAGE_TAG} \
                      ./backend/authService


                    echo "========================================"
                    echo "Building streaming service"
                    echo "========================================"

                    docker build \
                      -f ./backend/streamingService/Dockerfile \
                      -t ${ECR_REGISTRY}/streaming-service:${IMAGE_TAG} \
                      ./backend


                    echo "========================================"
                    echo "Building admin service"
                    echo "========================================"

                    docker build \
                      -f ./backend/adminService/Dockerfile \
                      -t ${ECR_REGISTRY}/streaming-admin:${IMAGE_TAG} \
                      ./backend


                    echo "========================================"
                    echo "Building chat service"
                    echo "========================================"

                    docker build \
                      -f ./backend/chatService/Dockerfile \
                      -t ${ECR_REGISTRY}/streaming-chat:${IMAGE_TAG} \
                      ./backend


                    echo "========================================"
                    echo "Building frontend"
                    echo "========================================"

                    docker build \
                      --build-arg REACT_APP_AUTH_API_URL=/api \
                      --build-arg REACT_APP_STREAMING_API_URL=/api \
                      --build-arg REACT_APP_STREAMING_PUBLIC_URL=/ \
                      --build-arg REACT_APP_ADMIN_API_URL=/api/admin \
                      --build-arg REACT_APP_CHAT_API_URL=/api/chat \
                      --build-arg REACT_APP_CHAT_SOCKET_URL=/ \
                      -t ${ECR_REGISTRY}/streaming-frontend:${IMAGE_TAG} \
                      ./frontend


                    echo "========================================"
                    echo "All five images built successfully."
                    echo "Image tag: ${IMAGE_TAG}"
                    echo "========================================"
                '''
            }
        }

        stage('Push Images') {
            steps {
                sh '''
                    set -e

                    echo "Pushing authentication image..."
                    docker push \
                      ${ECR_REGISTRY}/streaming-auth:${IMAGE_TAG}


                    echo "Pushing streaming image..."
                    docker push \
                      ${ECR_REGISTRY}/streaming-service:${IMAGE_TAG}


                    echo "Pushing admin image..."
                    docker push \
                      ${ECR_REGISTRY}/streaming-admin:${IMAGE_TAG}


                    echo "Pushing chat image..."
                    docker push \
                      ${ECR_REGISTRY}/streaming-chat:${IMAGE_TAG}


                    echo "Pushing frontend image..."
                    docker push \
                      ${ECR_REGISTRY}/streaming-frontend:${IMAGE_TAG}


                    echo "========================================"
                    echo "All five images pushed successfully."
                    echo "========================================"
                '''
            }
        }
    }

    post {
        success {
            echo 'StreamingApp CI/CD pipeline completed successfully.'
            echo "Images pushed with tag: ${IMAGE_TAG}"
        }

        failure {
            echo 'StreamingApp CI/CD pipeline failed.'
            echo "Check the Console Output for the failed stage."
        }

        always {
            echo "Build number: ${BUILD_NUMBER}"
            echo "ECR registry: ${ECR_REGISTRY}"
            echo "Image tag: ${IMAGE_TAG}"
        }
    }
}

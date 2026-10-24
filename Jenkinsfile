pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        AWS_ACCOUNT_ID = '994185519859'
        AWS_REGION     = 'us-east-1'
        ECR_REGISTRY   = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG      = "${BUILD_NUMBER}"

        // These paths are case-sensitive
        HELLO_PATH     = './backend/helloService'
        PROFILE_PATH   = './backend/profileService'
        FRONTEND_PATH  = './frontend'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Validate Repository') {
            steps {
                sh '''
                    set -e

                    echo "Workspace: $(pwd)"

                    test -f "$HELLO_PATH/Dockerfile" || {
                        echo "Missing Dockerfile: $HELLO_PATH/Dockerfile"
                        exit 1
                    }

                    test -f "$PROFILE_PATH/Dockerfile" || {
                        echo "Missing Dockerfile: $PROFILE_PATH/Dockerfile"
                        exit 1
                    }

                    test -f "$FRONTEND_PATH/Dockerfile" || {
                        echo "Missing Dockerfile: $FRONTEND_PATH/Dockerfile"
                        exit 1
                    }

                    echo "All Dockerfiles found successfully:"
                    echo " - $HELLO_PATH/Dockerfile"
                    echo " - $PROFILE_PATH/Dockerfile"
                    echo " - $FRONTEND_PATH/Dockerfile"
                '''
            }
        }

        stage('Check AWS Identity') {
            steps {
                sh '''
                    set -e

                    echo "Checking AWS identity..."
                    aws --version
                    aws sts get-caller-identity
                '''
            }
        }

        stage('AWS ECR Authentication') {
            steps {
                sh '''
                    set -e

                    echo "Authenticating with Amazon ECR..."

                    aws ecr get-login-password --region "$AWS_REGION" |
                        docker login \
                            --username AWS \
                            --password-stdin "$ECR_REGISTRY"

                    echo "ECR login successful"
                '''
            }
        }

        stage('Build & Push Helloservice') {
            steps {
                sh '''
                    set -e

                    IMAGE="$ECR_REGISTRY/helloservice:$IMAGE_TAG"

                    echo "Building Hello Service image:"
                    echo "$IMAGE"

                    docker build \
                        --tag "$IMAGE" \
                        "$HELLO_PATH"

                    echo "Pushing Hello Service image..."
                    docker push "$IMAGE"
                '''
            }
        }

        stage('Build & Push Profileservice') {
            steps {
                sh '''
                    set -e

                    IMAGE="$ECR_REGISTRY/profileservice:$IMAGE_TAG"

                    echo "Building Profile Service image:"
                    echo "$IMAGE"

                    docker build \
                        --tag "$IMAGE" \
                        "$PROFILE_PATH"

                    echo "Pushing Profile Service image..."
                    docker push "$IMAGE"
                '''
            }
        }

        stage('Build & Push Frontend') {
            steps {
                sh '''
                    set -e

                    IMAGE="$ECR_REGISTRY/frontend:$IMAGE_TAG"

                    echo "Building Frontend image:"
                    echo "$IMAGE"

                    docker build \
                        --tag "$IMAGE" \
                        "$FRONTEND_PATH"

                    echo "Pushing Frontend image..."
                    docker push "$IMAGE"
                '''
            }
        }
    }

    post {
        always {
            echo 'Cleaning up Docker resources...'

            sh '''
                docker logout "$ECR_REGISTRY" || true
                docker image prune -f || true
            '''
        }

        success {
            echo "Pipeline completed successfully."
            echo "Images were pushed with tag: ${IMAGE_TAG}"
        }

        failure {
            echo 'Pipeline failed. Review the build output.'
        }
    }
}

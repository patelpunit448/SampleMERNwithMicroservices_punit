pipeline {
    agent any

    options {
        // Prevent Jenkins from checking out the repository twice.
        skipDefaultCheckout(true)

        timestamps()
    }

    environment {
        AWS_ACCOUNT_ID = '994185519859'
        AWS_REGION     = 'us-east-1'
        ECR_REGISTRY   = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
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

        stage('Inspect Repository') {
    steps {
        sh '''
            set -e

            echo "Workspace:"
            pwd

            echo "Top-level directories:"
            find . -maxdepth 3 -type d | sort

            echo "Dockerfiles:"
            find . -type f -iname 'Dockerfile' -print
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
                '''
            }
        }

        stage('Build & Push Helloservice') {
            steps {
                sh '''
                    set -e

                    IMAGE="$ECR_REGISTRY/helloservice:$IMAGE_TAG"

                    echo "Building Hello Service: $IMAGE"
                    docker build \
                        -t "$IMAGE" \
                        ./backend/helloservice

                    echo "Pushing Hello Service..."
                    docker push "$IMAGE"
                '''
            }
        }

        stage('Build & Push Profileservice') {
            steps {
                sh '''
                    set -e

                    IMAGE="$ECR_REGISTRY/profileservice:$IMAGE_TAG"

                    echo "Building Profile Service: $IMAGE"
                    docker build \
                        -t "$IMAGE" \
                        ./backend/profileservice

                    echo "Pushing Profile Service..."
                    docker push "$IMAGE"
                '''
            }
        }

        stage('Build & Push Frontend') {
            steps {
                sh '''
                    set -e

                    IMAGE="$ECR_REGISTRY/frontend:$IMAGE_TAG"

                    echo "Building Frontend: $IMAGE"
                    docker build \
                        -t "$IMAGE" \
                        ./frontend

                    echo "Pushing Frontend..."
                    docker push "$IMAGE"
                '''
            }
        }
    }

    post {
        always {
            echo 'Cleaning up unused local Docker images...'

            sh '''
                docker image prune -f || true
            '''
        }

        success {
            echo "CI Pipeline execution completed successfully!"
            echo "Images pushed with tag: ${IMAGE_TAG}"
        }

        failure {
            echo "Pipeline failed. Review the build output."
        }
    }
}

pipeline {
    agent any
    
    environment {
        AWS_ACCOUNT_ID = '994185519859' // REPLACE WITH YOUR AWS ACCOUNT ID
        AWS_REGION     = 'us-east-1'           // REPLACE WITH YOUR ECR REGION
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        
        stage('AWS ECR Authentication') {
            steps {
                script {
                    echo "Authenticating with Amazon ECR..."
                    // Uses the EC2 Instance Profile IAM role automatically
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                }
            }
        }
        
        stage('Build & Push Helloservice') {
            steps {
                script {
                    echo "Building Hello Service..."
                    sh "docker build -t helloservice:${IMAGE_TAG} ./backend/helloservice"
                    sh "docker tag helloservice:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}://"
                    sh "docker tag helloservice:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}://{IMAGE_TAG}"
                    
                    echo "Pushing Hello Service to ECR..."
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}://"
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}://{IMAGE_TAG}"
                }
            }
        }
        
        stage('Build & Push Profileservice') {
            steps {
                script {
                    echo "Building Profile Service..."
                    sh "docker build -t profileservice:${IMAGE_TAG} ./backend/profileservice"
                    sh "docker tag profileservice:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}://"
                    sh "docker tag profileservice:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}://{IMAGE_TAG}"
                    
                    echo "Pushing Profile Service to ECR..."
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}://"
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}://{IMAGE_TAG}"
                }
            }
        }
        
        stage('Build & Push Frontend') {
            steps {
                script {
                    echo "Building Frontend..."
                    sh "docker build -t frontend:${IMAGE_TAG} ./frontend"
                    sh "docker tag frontend:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}://"
                    sh "docker tag frontend:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}://{IMAGE_TAG}"
                    
                    echo "Pushing Frontend to ECR..."
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}://"
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}://{IMAGE_TAG}"
                }
            }
        }
    }
    
    post {
        always {
            echo "Cleaning up local workspace images to preserve server disk space..."
            sh "docker image prune -f"
        }
        success {
            echo "CI Pipeline execution completed successfully!"
        }
        failure {
            echo "Pipeline failed. Review build steps for structural mismatches."
        }
    }
}

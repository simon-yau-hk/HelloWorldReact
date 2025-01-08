pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'docker.io'  // Replace with your registry
        DOCKER_IMAGE = 'your-app-name'
        DOCKER_TAG = "${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = 'docker-cred-id'  // Jenkins credentials ID
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Get code from repository
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    docker.build("${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        
        stage('Push to Registry') {
            steps {
                script {
                    // Login to Docker registry and push image
                    docker.withRegistry("https://${DOCKER_REGISTRY}", DOCKER_CREDENTIALS_ID) {
                        docker.image("${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        // Also push as 'latest'
                        docker.image("${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')
                    }
                }
            }
        }
        
        stage('Clean Up') {
            steps {
                // Remove local Docker images
                sh "docker rmi ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                sh "docker rmi ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest"
            }
        }
    }
    
    post {
        always {
            // Clean workspace
            cleanWs()
        }
    }
}
pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'my-web-app'
        DOCKER_USERNAME = 'simonyauwl'
        DOCKER_CREDENTIALS_ID = 'docker-cred-id'  // Jenkins credentials ID
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Get code from repository
                checkout scm
            }
        }

        stage('Login Docker') {
            steps {
                script {
                   // Login to Docker Hub
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    }

                }
            }
        }
        
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                script {
                    // Tag the Docker image
                    sh "docker tag ${DOCKER_IMAGE} ${DOCKER_USERNAME}/${DOCKER_IMAGE}"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // Push the tagged image to Docker Hub
                    sh "docker push  ${DOCKER_USERNAME}/${DOCKER_IMAGE}"
                }
            }
        }

        
        stage('Clean Up') {
            steps {
                // Remove local Docker images
                sh "docker rmi  ${DOCKER_USERNAME}/${DOCKER_IMAGE}:latest"
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
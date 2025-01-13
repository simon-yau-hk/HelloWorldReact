pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'my-web-app'
        DOCKER_USERNAME = 'simonyauwl'
        DOCKER_CREDENTIALS_ID = 'docker-cred-id'  // Jenkins credentials ID
         // Add Git tag/commit environment variables
        NEW_TAG = ''
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Get code from repository
                checkout scm
            }
        }

        stage('Create New Tag') {
            steps {
                script {
                    echo "Repository URL: \$(git config --get remote.origin.url)"
                    echo "Current branch: \$(git branch --show-current)"
                    // First, make sure we fetch all tags
                    sh "git fetch --tags"
                    // Get the latest tag
                    def latestTag = sh(returnStdout: true, script: 'git describe --tags --abbrev=0').trim()
                    
                    // Parse version numbers
                    def (major, minor, patch) = latestTag.replaceAll('v', '').tokenize('.')
                    
                    // Increment patch version
                    def newPatch = patch.toInteger() + 1
                    def newTag = "v${major}.${minor}.${newPatch}"
                    
                      // Store the new tag for later use
                    env.NEW_TAG = newTag

                    // Create and push new tag
                    // git config --global user.email "simon_yau@hotmail.com.hk"
                    // git config --global user.name "simon-yau-hk"
                    sh """
                        git tag ${newTag}
                        git push origin ${newTag}
                    """
                    
                    echo "Created new tag: ${newTag}"
                }
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
                    // Tag with Git version
                    sh "docker tag ${DOCKER_IMAGE} ${DOCKER_USERNAME}/${DOCKER_IMAGE}:${NEW_TAG}"
                    // Tag the Docker image
                    sh "docker tag ${DOCKER_IMAGE} ${DOCKER_USERNAME}/${DOCKER_IMAGE}"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // Push the tagged image to Docker Hub
                    sh "docker push ${DOCKER_USERNAME}/${DOCKER_IMAGE}:${NEW_TAG}"
                    sh "docker push  ${DOCKER_USERNAME}/${DOCKER_IMAGE}"
                }
            }
        }

        
        stage('Clean Up') {
            steps {
                // Remove local Docker images
                sh "docker rmi ${DOCKER_USERNAME}/${DOCKER_IMAGE}:${NEW_TAG}"
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
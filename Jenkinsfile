pipeline {
    agent any 

    environment {
        // Define environment variables
        GIT_REPO_URL = 'https://github.com/FaithKadiri/Django.git' // Your GitHub repo URL
        BRANCH_NAME = 'main' // Change to your target branch if needed
        DOCKER_IMAGE_NAME = 'faithkadiri/django:latest' // Docker image name
        AWS_INSTANCE_IP = '18.118.9.15' // Your AWS instance's public IP
        SSH_KEY_PATH = credentials('faith-aws-ssh-key') // Jenkins credentials ID for your SSH key
        DOCKER_USERNAME = 'faithkadiri' // Your Docker Hub username
        DOCKER_PASSWORD = credentials('dockerhub-credentials') // Jenkins credentials ID for Docker password
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the specified branch
                    git branch: BRANCH_NAME, url: GIT_REPO_URL
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub
                    echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    docker.build(DOCKER_IMAGE_NAME)
                }
            }
        }

        stage('Deploy to AWS') {
            steps {
                script {
                    // Deploy the Docker container to AWS instance
                    sshagent (credentials: ['faith-aws-ssh-key']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${AWS_INSTANCE_IP} << EOF
                        docker pull ${DOCKER_IMAGE_NAME}
                        docker stop \$(docker ps -q --filter "ancestor=${DOCKER_IMAGE_NAME}") || true
                        docker run -d -p 80:8000 ${DOCKER_IMAGE_NAME}
                        EOF
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully.'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}

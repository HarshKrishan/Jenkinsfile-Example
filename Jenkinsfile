pipeline {
    agent any
    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials' // Jenkins credentials ID for Docker Hub
        GIT_CREDENTIALS_ID = 'github-credentials'       // Jenkins credentials ID for GitHub
        DOCKER_IMAGE_NAME = 'your-dockerhub-username/your-react-app'
        IMAGE_VERSION = "${env.BUILD_ID}"               // Unique tag for each build
        GITHUB_REPO = 'https://github.com/your-username/your-repo.git'  // Your GitHub repo URL
    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout code from GitHub
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                              userRemoteConfigs: [[url: "${GITHUB_REPO}",
                                                   credentialsId: "${env.GIT_CREDENTIALS_ID}"]]])
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image using the Dockerfile in the repository
                    dockerImage = docker.build("${DOCKER_IMAGE_NAME}:${IMAGE_VERSION}")
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh 'echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin'
                    }
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Push the Docker image to Docker Hub
                    dockerImage.push('latest')
                    dockerImage.push("${IMAGE_VERSION}")
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    // Cleanup local Docker images to free up space
                    sh "docker rmi ${DOCKER_IMAGE_NAME}:${IMAGE_VERSION}"
                }
            }
        }
    }

    post {
        always {
            script {
                // Ensure Docker logout happens even if build fails
                sh 'docker logout'
            }
        }
    }
}

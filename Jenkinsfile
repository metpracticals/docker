pipeline {
    agent any

    environment {
        // Replace with your Docker Hub username and repository name
        DOCKER_HUB_USER = 'docksep30'
        IMAGE_NAME      = 'html-app'
        IMAGE_TAG       = "${BUILD_NUMBER}"
        // The ID of the credentials you save in Jenkins for Docker Hub
        DOCKER_HUB_CREDS = 'docker-hub-credentials-id'
    }

    stages {
        // STAGE 1: Pull latest code from Git
        stage('Pull Code') {
            steps {
                // 'checkout scm' automatically pulls the exact commit that triggered the build
                checkout scm
            }
        }

        // STAGE 2: Run unit tests and display results
        stage('Unit Tests') {
            steps {
                script {
                    // Installs htmlhint locally and runs it against all HTML files
                    sh 'npm install htmlhint'
                    sh './node_modules/.bin/htmlhint "**/*.html"'
                }
            }
        }

        // STAGE 3: Build a Docker image from the project
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "docker tag ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        // STAGE 4: Push the docker image to Docker Hub
        stage('Push to Docker Hub') {
            steps {
                script {
                    // Securely logs into Docker Hub using Jenkins Credentials Provider
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDS}", usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh "echo '${PASS}' | docker login -u '${USER}' --password-stdin"
                        sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up local Docker images..."
            sh "docker rmi ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true"
            sh "docker rmi ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest || true"
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the console output above."
        }
    }
}

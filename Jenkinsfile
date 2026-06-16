pipeline {
    agent any

        environment {
        // Explicitly defining the global public endpoint
        DOCKER_HUB_USER  = 'docksep30' 
        IMAGE_NAME       = 'html-app'
        IMAGE_TAG        = "${BUILD_NUMBER}"
        DOCKER_HUB_CREDS = 'docker-hub-credentials-id'
    }


    stages {
        // STAGE 1: Pull latest code from Git
        stage('Pull Code') {
            steps {
                checkout scm
            }
        }

        // STAGE 2: Run unit tests and display results
        stage('Unit Tests') {
            steps {
                script {
                    bat 'npm install htmlhint'
                    bat 'node_modules\\.bin\\htmlhint "**/*.html"'
                }
            }
        }

        // STAGE 3: Build a Docker image from the project
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    bat "docker build -t %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG% ."
                    bat "docker tag %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG% %DOCKER_HUB_USER%/%IMAGE_NAME%:latest"
                }
            }
        }

        // STAGE 4: Push the docker image to Docker Hub
        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDS}", usernameVariable: 'DUSER', passwordVariable: 'DPASS')]) {
                        // Quotes securely wrap special characters on Windows CMD
                        bat 'docker login -u "%DUSER%" -p "%DPASS%"'
                        bat "docker push %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG%"
                        bat "docker push %DOCKER_HUB_USER%/%IMAGE_NAME%:latest"
                    }
                }
            }
        } // Stage 4 ends here cleanly
    } // Stages block ends here cleanly

    post {
        always {
            echo "Cleaning up local Docker images..."
            bat "docker rmi %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG% || exit 0"
            bat "docker rmi %DOCKER_HUB_USER%/%IMAGE_NAME%:latest || exit 0"
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the console output above."
        }
    }
}

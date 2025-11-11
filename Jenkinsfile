pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "my-web-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                bat """
                    docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% .
                    docker tag %DOCKER_IMAGE%:%BUILD_NUMBER% %DOCKER_IMAGE%:latest
                """
            }
        }
        
        stage('Test Image') {
            steps {
                echo 'Testing Docker image...'
                bat 'docker images | findstr %DOCKER_IMAGE%'
            }
        }
        
        stage('Run Container') {
            steps {
                echo 'Running container for testing...'
                bat """
                    docker rm -f test-container >nul 2>&1
                    docker run -d --name test-container -p 8081:80 %DOCKER_IMAGE%:%BUILD_NUMBER%
                    timeout /t 3
                    curl -f http://localhost:8081 || exit 1
                    echo Container is running successfully!
                """
            }
        }
        
        stage('Cleanup') {
            steps {
                echo 'Cleaning up test container...'
                bat """
                    docker stop test-container >nul 2>&1
                    docker rm test-container >nul 2>&1
                """
            }
        }
    }
    
    post {
        success {
            echo "✅ Pipeline completed! Docker image ${DOCKER_IMAGE}:${DOCKER_TAG} is ready!"
        }
        failure {
            echo '❌ Pipeline failed!'
            bat """
                docker stop test-container >nul 2>&1
                docker rm test-container >nul 2>&1
            """
        }
    }
}

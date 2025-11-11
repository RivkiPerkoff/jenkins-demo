pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "my-web-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
        PORT = "8081"
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
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }
        
        stage('Run Container') {
            steps {
                echo 'Running container for testing...'
                sh """
                    # אם קונטיינר קיים – עצור והסר
                    docker rm -f test-container 2>/dev/null || true
                    docker run -d --name test-container -p ${PORT}:80 ${DOCKER_IMAGE}:${DOCKER_TAG}
                    sleep 3
                    curl -f http://localhost:${PORT} || exit 1
                    echo "Container is running successfully!"
                """
            }
        }
        
        stage('Cleanup') {
            steps {
                echo 'Cleaning up test container...'
                sh """
                    docker stop test-container 2>/dev/null || true
                    docker rm test-container 2>/dev/null || true
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
            sh """
                docker stop test-container 2>/dev/null || true
                docker rm test-container 2>/dev/null || true
            """
        }
    }
}

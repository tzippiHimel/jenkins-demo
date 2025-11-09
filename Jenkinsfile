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
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
            }
        }
        
        stage('Test Image Exists') {
            steps {
                echo 'Testing Docker image...'
                sh "docker images | grep ${DOCKER_IMAGE}"
            }
        }
        
        stage('Run Container For Test') {
            steps {
                echo 'Running container for testing...'
                sh '''
                    docker rm -f test-container || true
                    docker run -d --name test-container -p 8081:80 ${DOCKER_IMAGE}:${DOCKER_TAG}
                    sleep 5
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Checking container health...'
                sh '''
                    docker exec test-container sh -c "apk add curl >/dev/null 2>&1 || true"
                    docker exec test-container curl -f http://localhost || exit 1
                    echo "Container responded OK ✅"
                '''
            }
        }
        
        stage('Cleanup') {
            steps {
                echo 'Cleaning up test container...'
                sh "docker stop test-container || true"
                sh "docker rm test-container || true"
            }
        }
    }
    
    post {
        success {
            echo "✅ Pipeline completed! Docker image ${DOCKER_IMAGE}:${DOCKER_TAG} is ready!"
        }
        failure {
            echo "❌ Pipeline failed!"
            sh "docker stop test-container || true"
            sh "docker rm test-container || true"
        }
    }
}

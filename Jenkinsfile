pipeline {
    agent any
    tools {
        maven 'mvn3.9.9' 
    }
  environment {
    DOCKER_HUB_USER  = 'bharanitharan9090'
    DOCKER_IMAGE    = 'bharanitharan9090/simplybyte-calculator-jenkins-local'
    CONTAINER_NAME  = 'simplybyte-container-calculator'
    
    // Changed from 'docker-hub-token' to 'docker' to match your locked Jenkins ID
    DOCKER_HUB_TOKEN = credentials('docker') 
}

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['testing','prod'], description: 'Select deployment environment')
    }
    stages {
        stage('print path') {
            steps {
                bat 'echo %PATH%'
            }
        }
        stage('maven test stage') {
            when {
                expression { params.ENVIRONMENT == 'testing' }
            }
            steps {
                bat 'mvn test'
            }
        }
        stage('maven build stage') {
            steps {
                bat 'mvn clean install -DskipTests=true'
            }
        }
        stage('Docker image build') {
            steps {
                // Generates the tagged image ready for Docker Hub
                bat 'docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% .'
            }
        }
        
        stage('Docker Push to Hub') {
            steps {
                // Securely logs into Docker Hub using the env token and pushes the image
                bat 'echo %DOCKER_HUB_TOKEN% | docker login -u %DOCKER_HUB_USER% --password-stdin'
                bat 'docker push %DOCKER_IMAGE%:%BUILD_NUMBER%'
                bat 'docker logout'
            }
        }

        stage('Docker Local Run (Testing)') {
            when {
                expression { params.ENVIRONMENT == 'testing' }
            }
            steps {
                bat 'docker rm -f %CONTAINER_NAME% 2>nul || exit 0' 
                bat 'docker run -d -p 8091:8090 --name=%CONTAINER_NAME% %DOCKER_IMAGE%:%BUILD_NUMBER%'
            }
        }

        stage('Deploy to Production Server') {
            when {
                expression { params.ENVIRONMENT == 'prod' }
            }
            steps {
                echo "Deploying Build #%BUILD_NUMBER% to the live production server..."
                // NOTE: Replace 'prod-server-ip' with your actual production machine IP address
                // Using SSH to log into production, login to docker, pull down, and host the image
                bat '''
                ssh user@prod-server-ip "echo %DOCKER_HUB_TOKEN% | docker login -u %DOCKER_HUB_USER% --password-stdin && docker pull %DOCKER_IMAGE%:%BUILD_NUMBER% && docker rm -f %CONTAINER_NAME% 2>/dev/null || true && docker run -d -p 8091:8090 --name=%CONTAINER_NAME% --restart=unless-stopped %DOCKER_IMAGE%:%BUILD_NUMBER%"
                '''
            }
        }
    }
}

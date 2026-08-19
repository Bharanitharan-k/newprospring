pipeline {
    agent any
    tools {
        maven 'mvn3.9.9' 
    }
    environment {
        DOCKER_IMAGE    = 'bharanitharan9090/simplybyte-calculator-jenkins-local'
        CONTAINER_NAME  = 'simplybyte-container-calculator'
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
                bat "docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% ."
            }
        }
        
        stage('Docker Push to Hub') {
            steps {
                // withCredentials securely injects the username and password from the 'docker' ID
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat "echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin"
                    bat "docker push %DOCKER_IMAGE%:%BUILD_NUMBER%"
                    bat "docker logout"
                }
            }
        }

        stage('Docker Local Run (Testing)') {
            when {
                expression { params.ENVIRONMENT == 'testing' }
            }
            steps {
                bat 'docker rm -f %CONTAINER_NAME% 2>nul || exit 0' 
                bat "docker run -d -p 8091:8090 --name=%CONTAINER_NAME% %DOCKER_IMAGE%:%BUILD_NUMBER%"
            }
        }

        stage('Deploy to Production Server') {
            when {
                expression { params.ENVIRONMENT == 'prod' }
            }
            steps {
                echo "Deploying Build #%BUILD_NUMBER% to the live production server..."
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    // String interpolation via double quotes lets Jenkins pass the variables to Windows batch
                    bat """
                    ssh user@prod-server-ip "echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin && docker pull %DOCKER_IMAGE%:%BUILD_NUMBER% && docker rm -f %CONTAINER_NAME% 2>/dev/null || true && docker run -d -p 8091:8090 --name=%CONTAINER_NAME% --restart=unless-stopped %DOCKER_IMAGE%:%BUILD_NUMBER%"
                    """
                }
            }
        }
    }
}

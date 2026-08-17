pipeline {
    agent any
    tools {
        maven 'mvn3.9.9' 
    }
    environment {
        DOCKER_IMAGE='simplybyte-calculator-jenkins-local'
        CONTAINER_NAME='simplybyte-container-calculator'
    }
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['testing','prod'], description: 'environment')
    }
    stages {
        stage('print path') {
            steps {
                bat 'echo %PATH%' // Changed sh to bat, and syntax to Windows env
            }
        }
        stage('maven test stage') {
            when {
                expression {
                    params.ENVIRONMENT == 'testing'
                }
            }
            steps {
                bat 'mvn test' // Changed sh to bat
            }
        }
        stage('maven build stage') {
            steps {
                bat 'mvn clean install -DskipTests=true' // Changed sh to bat
            }
        }
        stage('Docker image build') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% .' // Changed sh to bat, using % env format
            }
        }
        stage('Docker old container remove') {
            steps {
                // In Windows CMD, we use "|| rem" or wrap in a try block to ignore failures gracefully
                bat 'docker rm -f %CONTAINER_NAME% 2>nul || exit 0' 
            }
        }
        stage('Docker new container run') {
            steps {
                bat 'docker run -d -p 8091:8090 --name=%CONTAINER_NAME% %DOCKER_IMAGE%:%BUILD_NUMBER%'
            }
        }
    }
}

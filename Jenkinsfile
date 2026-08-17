pipeline {
    agent any
    
    // Make sure you either configured 'mvn3.9.9' in Jenkins Tools 
    // OR removed this tools block to use your local './mvnw' wrapper.
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
        // REMOVED THE 'git clone' STAGE FROM HERE DIRECTLY
        
        stage('print path') {
            steps {
                sh 'echo $PATH'
            }
        }
        stage('maven test stage') {
            when {
                expression {
                    params.ENVIRONMENT == 'testing'
                }
            }
            steps {
                sh 'mvn test' // Use './mvnw test' if you didn't setup Jenkins global tools
            }
        }
        stage('maven build stage') {
            steps {
                sh 'mvn clean install -DskipTests=true' // Use './mvnw clean...' if using wrapper
            }
        }
        stage('Docker image build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .'
            }
        }
        stage('Docker old container remove') {
            steps {
                sh 'docker rm -f $CONTAINER_NAME || true'
            }
        }
        stage('Docker new container run') {
            steps {
                sh 'docker run -d -p 8091:8090 --name=$CONTAINER_NAME $DOCKER_IMAGE:$BUILD_NUMBER'
            }
        }
    }
}

pipeline { 
    agent any 
    
    tools { 
        maven 'mvn3.9.9' 
    } 
    
    environment { 
        DOCKER_IMAGE   = 'bharanitharan9090/simplybyte-calculator-jenkins-local' 
        CONTAINER_NAME = 'simplybyte-container-calculator' 
    } 
    
    parameters { 
        choice(name: 'ENVIRONMENT', choices: ['testing', 'prod'], description: 'Select deployment environment') 
    } 
    
    stages { 
        stage('Verify Environment') { 
            steps { 
                bat 'echo %PATH%' 
                bat 'mvn -v'
                bat 'docker -v'
            } 
        } 
        
        stage('Maven Test') { 
            when { 
                expression { params.ENVIRONMENT == 'testing' } 
            } 
            steps { 
                bat 'mvn test' 
            } 
        } 
        
        stage('Maven Build') { 
            steps { 
                bat 'mvn clean install -DskipTests=true' 
            } 
        } 
        
        stage('Docker Build') { 
            steps { 
                bat "docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% ." 
            } 
        } 
        
        stage('Docker Push to Hub') { 
            steps { 
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) { 
                    bat "echo %DOCKER_PASS%| docker login -u %DOCKER_USER% --password-stdin" 
                    bat "docker push %DOCKER_IMAGE%:%BUILD_NUMBER%" 
                    bat "docker logout" 
                } 
            } 
        } 
        
        stage('Local Deploy (Testing)') { 
            when { 
                expression { params.ENVIRONMENT == 'testing' } 
            } 
            steps { 
                bat "docker rm -f %CONTAINER_NAME% 2>nul || exit 0" 
                bat "docker run -d -p 8091:8090 --name=%CONTAINER_NAME% %DOCKER_IMAGE%:%BUILD_NUMBER%" 
            } 
        } 
        
        stage('Remote Deploy (Production)') { 
            when { 
                expression { params.ENVIRONMENT == 'prod' } 
            } 
            steps { 
                echo "Deploying Build #%BUILD_NUMBER% to the live production server..." 
                
                // Inject the IP, Docker Credentials, and native SSH private key file path
                withCredentials([
                    string(credentialsId: 'prod-server-ip', variable: 'PROD_IP'),
                    usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS'),
                    sshUserPrivateKey(credentialsId: 'prod-ssh-key', keyFileVariable: 'SSH_KEY_PATH', usernameVariable: 'SSH_USER')
                ]) { 
                    // Natively references the file path (%SSH_KEY_PATH%) via Windows command line
                    bat """
                    ssh -i "%SSH_KEY_PATH%" -o StrictHostKeyChecking=no %SSH_USER%@%PROD_IP% "echo %DOCKER_PASS%| docker login -u %DOCKER_USER% --password-stdin && docker pull %DOCKER_IMAGE%:%BUILD_NUMBER% && docker rm -f %CONTAINER_NAME% 2>/dev/null || true && docker run -d -p 8091:8090 --name=%CONTAINER_NAME% --restart=unless-stopped %DOCKER_IMAGE%:%BUILD_NUMBER%"
                    """
                } 
            } 
        } 
    } 
}

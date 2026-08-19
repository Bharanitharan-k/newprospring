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
                    // FIXED: Removed trailing space before the pipe (|) symbol to prevent broken Windows batch authentication strings
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
                // Windows syntax to clean up existing container safely without crashing if missing
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
                
                withCredentials([
                    string(credentialsId: 'prod-server-ip', variable: 'PROD_IP'),
                    usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')
                ]) { 
                    sshagent(credentials: ['prod-ssh-key']) { 
                        // FIXED: Removed trailing space before the pipe (|) symbol inside the nested inline remote execution script
                        bat """
                        ssh -o StrictHostKeyChecking=no user@%PROD_IP% "echo %DOCKER_PASS%| docker login -u %DOCKER_USER% --password-stdin && docker pull %DOCKER_IMAGE%:%BUILD_NUMBER% && docker rm -f %CONTAINER_NAME% 2>/dev/null || true && docker run -d -p 8091:8090 --name=%CONTAINER_NAME% --restart=unless-stopped %DOCKER_IMAGE%:%BUILD_NUMBER%"
                        """
                    } 
                } 
            } 
        } 
    } 
}

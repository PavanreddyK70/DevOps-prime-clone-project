pipeline {
    agent any
    
    tools {
        jdk 'JDK'
        nodejs 'NodeJS'
    }
    
    environment {
        SCANNER_HOME=tool 'SonarQube Scanner' 
    }
    stages {
        stage ('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/PavanreddyK70/DevOps-prime-clone-project.git'
            }
        }
        stage ('Sonarqube analysis') {
            steps {
                withSonarQubeEnv ('sonar-server') {
                    sh '''
                       $SCANNER_HOME/bin/sonar-scanner \
                       -Dsonar.projectName=amazon-prime \
                       -Dsonar.projectKey=amazon-prime \
                       -Dsonar.sourceEncoding=UTF-8 \
                       -Dsonar.scm.disabled=false
                    '''
                }
            }
        }
        stage ('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
        }
        stage ('NPM Install') {
            steps {
                sh 'npm install'
            }
        }
        stage ('Trivy Scan') {
            steps {
                sh 'trivy fs . > trivy-scan-results.txt'
            }
        }
        stage ('Build image') {
            steps {
                sh 'docker build -t prime-clone .'
            }
        }
        stage ('Tag and push') {
            steps {
                withDockerRegistry(url: 'https://index.docker.io/v1/', credentialsId: 'docker') {
                    sh '''
                       docker tag prime-clone dockerpavank/prime-clone:${BUILD_NUMBER}
                       docker tag prime-clone dockerpavank/prime-clone:latest
                       docker push dockerpavank/prime-clone:${BUILD_NUMBER}
                       docker push dockerpavank/prime-clone:latest
                       '''
                }
            }
        }
        stage('Cleanupimage from jenkins server') {
            steps {
                sh 'docker rmi dockerpavank/prime-clone:${BUILD_NUMBER} || true'
                sh 'docker rmi dockerpavank/prime-clone:latest || true'
            }
        }
    }
}

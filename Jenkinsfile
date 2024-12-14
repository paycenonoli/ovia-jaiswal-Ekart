pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-token', url: 'https://github.com/paycenonoli/ovia-jaiswal-Ekart.git'
            }
        }
        
        stage('Maven Compile') {
            steps {
                sh 'mvn clean compile -DskipTests=true'
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan target/', odcInstallation: 'owasp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
         stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Shopping-Cart \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Shopping-Cart '''
                   }

            }
        }
        
        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }
        
        stage('Docker Build') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'ovia-docker-cred', toolName: 'docker') {
                        sh 'docker build -t ovia-cart -f docker/Dockerfile .'   
                        sh 'docker tag ovia-cart ojpascale/shopping-cart:latest'
                        sh 'docker push ojpascale/shopping-cart:latest'
                    }
                }
            }
        }
    }
}

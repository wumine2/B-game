pipeline {
    agent any 
    
    tools{
        jdk 'jdk'
        maven 'maven'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    stages{
        
        stage("Git Checkout"){
            steps{
               git branch: 'main', credentialsId: 'git-token', url: 'https://github.com/wumine2/boardgame.git'
            }
        }
        
        stage("Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
        
        stage("Test"){
            steps{
                sh "mvn test"
            }
        }
        
        stage("File System Scan"){
            steps{
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner 
                    -Dsonar.projectName=boardgame \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=boardgame '''
    
                }
            }
            
        stage('Hello') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                         sh "mvn deploy"
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                    sh " docker build -t wumine2/boardgame:v1 ."
                }
            }
        }
        
        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-fs-report.html wumine2/boardgame:v1"
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package"
            }
        }

        }
    }   
}
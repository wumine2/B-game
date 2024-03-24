pipeline {
    agent any 
    
    tools {
        jdk 'jdk'
        maven 'maven'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-token', url: 'https://github.com/wumine2/boardgame.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Test'){
            steps{
                sh "mvn test"
            }
        }
        
        stage('File System Scan'){
            steps{
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage('Sonarqube Analysis'){
            steps{
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=boardgame \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=boardgame '''
                }
            }
        }
        
                
        stage('Build Package') {
            steps {
                sh "mvn package"
            }
        }
        
        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                     sh "mvn deploy"
            }
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                    sh " docker build -t wumine2/boardgame:v1 ."
               }
            }
        }
        }
        
        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-fs-report.html wumine2/boardgame:v1"
            }
        }
        
        stage('Docker Image Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                    sh " docker push wumine2/boardgame:v1"
            }
        }
     }
    }

     stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "argocd-demo"
            GIT_USER_NAME = "wumine2"
        }
        steps {
            withCredentials([string(credentialsId: 'git-token', variable: 'git-token')]) {
                sh '''
                    git config user.email "atiwunmi@yahoo.com"
                    git config user.name "Wunmi"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" prod/pod.yaml
                    git add prod/pod.yaml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${git-token}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
                 }
            }
}
}
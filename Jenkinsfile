pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        maven 'maven2'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    

    stages {
        
        stage('Git Checkout ') {
            steps {
               git branch: 'main', url: 'https://github.com/kiki092002/BoardgameListingWebApp'
            }
        }
        
        stage('Compile') {
            steps {
               sh 'mvn compile'
            }
        }
        
        stage('Test') {
            steps {
               sh 'mvn test'
            }
        }
        
        stage('File System Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        
        stage('Sonar Analysis') {
            steps {
               withSonarQubeEnv('sonar'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=OnlineGame \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=OnlineGame '''
               }
            }
        }
        
     
        
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven2', mavenSettingsConfig: '', traceability: true) {
                     sh 'mvn deploy'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred',toolName: 'docker') {
                    sh "docker build -t kiki092002/boardgame:latest ."
                 }
               }
            }
        }
        stage('Docker Image Scan') {
            steps {
                sh 'trivy image --format table -o trivy-fs-report1.html kiki092002/boardgame:latest'
            }
        }
        
        stage('Docker Push') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred',toolName: 'docker') {
                    sh "docker push kiki092002/boardgame:latest"
                 }
               }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f deployment-service.yaml'
            }
        }
        
        stage('Verify Deployment') {
            steps {
               sh 'kubectl get pods'
               sh 'kubectl get svc'
            }
        }
    }
}


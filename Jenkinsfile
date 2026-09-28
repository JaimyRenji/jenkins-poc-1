pipeline { 
    agent any
    environment{
        CI='true'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build'){
            steps{
                sh 'npm install'
                sh 'npm run build'
            }
       }
       stage('Test'){
            steps{
                sh './jenkins/scripts/test.sh'
            }
       }
       stage('SonarQube Analysis') {
          steps {
             withSonarQubeEnv('Sonarqube') {
            sh 'sonar-scanner'
            }
       }
}
}
}

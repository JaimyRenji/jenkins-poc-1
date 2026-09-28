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
                 script {
                def scannerHome = tool 'sonar-scanner'
                sh "${scannerHome}/bin/sonar-scanner"
            }
            }
       }
}
    stage('Quality Gate') {
    steps {
        timeout(time: 5, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
}
}

pipeline {
    agent any

    environment {
        CI = 'true'
        NEXUS_URL = 'http://3.111.147.229:8081/repository/jenkins-poc'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
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

stage('Package Feature') {
    when {
        branch 'feature-branch-1'
    }
    steps {
        sh 'tar -czf myapp-feature-branch-1-${BUILD_NUMBER}.tar.gz build/'
    }
}

stage('Upload Feature Artifact') {
    when {
        branch 'feature-branch-1'
    }
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'nexus-credentials',
                usernameVariable: 'NEXUS_USER',
                passwordVariable: 'NEXUS_PASSWORD'
            )
        ]) {
            sh '''
                curl -u "$NEXUS_USER:$NEXUS_PASSWORD" \
                --upload-file "myapp-feature-branch-1-${BUILD_NUMBER}.tar.gz" \
                "${NEXUS_URL}/myapp-feature-branch-1-${BUILD_NUMBER}.tar.gz"
            '''
        }
    }
}


        stage('Package Release') {
            when {
                branch 'master'
            }

            steps {
                sh 'tar -czf myapp-release-${BUILD_NUMBER}.tar.gz build/'
            }
        }

        stage('Upload Release Artifact') {
            when {
                branch 'master'
            }

            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-credentials',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )
                ]) {
                    sh '''
                        curl -u "$NEXUS_USER:$NEXUS_PASSWORD" \
                        --upload-file "myapp-release-${BUILD_NUMBER}.tar.gz" \
                        "${NEXUS_URL}/myapp-release-${BUILD_NUMBER}.tar.gz"
                    '''
                }
            }
        }
stage('Download Release from Nexus') {
    when {
        branch 'master'
    }
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'nexus-credentials',
                usernameVariable: 'NEXUS_USER',
                passwordVariable: 'NEXUS_PASSWORD'
            )
        ]) {
            sh '''
                curl -f -u "$NEXUS_USER:$NEXUS_PASSWORD" \
                -o "myapp-release-${BUILD_NUMBER}.tar.gz" \
                "${NEXUS_URL}/myapp-release-${BUILD_NUMBER}.tar.gz"
            '''
        }
    }
}
    stage('Copy Artifact to EC2') {
    when {
        branch 'master'
    }
    steps {
        sshagent(['deploy-ec2']) {
            sh '''
                ssh -o StrictHostKeyChecking=no \
                ec2-user@3.111.147.229 \
                "mkdir -p /opt/myapp"

                scp -o StrictHostKeyChecking=no \
                "myapp-release-${BUILD_NUMBER}.tar.gz" \
                ec2-user@3.111.147.229:/opt/myapp/
            '''
        }
    }
}
        stage('Smoke Test') {
    when {
        branch 'master'
    }
    steps {
        sshagent(['deploy-ec2']) {
            sh '''
                ssh -o StrictHostKeyChecking=no \
                ec2-user@3.111.147.229 \
                "curl -f http://localhost:3000"
            '''
        }
    }
}
    }
}

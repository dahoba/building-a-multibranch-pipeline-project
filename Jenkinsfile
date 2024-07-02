pipeline {
    agent any
    environment{
        CI = 'true'
    }
    stages {
        stage('Build') {
            steps {
              nodejs(nodeJSInstallationName: 'Node 20.x'){
                sh 'npm install'
              } 
            }
        }
        stage('Test'){
            steps{
                sh './jenkins/scripts/test.sh'
            }
        }
    }
}

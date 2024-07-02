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
                nodejs(nodeJSInstallationName: 'Node 20.x'){
                  sh './jenkins/scripts/test.sh'
                }
            }
        }
        stage('Deliver for develop'){
            when{
                branch 'develop'
            }
            steps{
                nodejs(nodeJSInstallationName: 'Node 20.x'){
                  sh './jenkins/scripts/deliver-for-development.sh'
                  input message: 'Finished using the web site? (Click "Proceed" to continue)'
                  sh './jenkins/scripts/kill.sh'
                }
            }
        }
        stage('Deploy for production'){
            when{
                branch 'master'
            }
            steps{
              sh './jenkins/scripts/deliver-for-production.sh'
              input message: 'Finished using the web site? (Click "Proceed" to continue)'
              sh './jenkins/scripts/kill.sh'

            }
        }
    }
}

pipeline {
    agent any
    options{
      buildDiscarder(logRotator(numToKeepStr: '1'))
      disableConcurrentBuilds()
    }
    environment{
      CI = 'true'
      COMMIT_SHORT_SHA = """${sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()}"""
      DOCKER_REGISTRY = 'ghcr.io/dahoba'
      DOCKER_IMAGE_NAME = 'building-a-multibranch-pipeline-project'
      DOCKER_IMAGE_TAG = '${COMMIT_SHORT_SHA}'
      REPO_NAME = 'building-a-multibranch-pipeline-project'
      DEBUG_MODE = 'false'
      // https://github.com/dahoba/building-a-multibranch-pipeline-project
    }
    stages {
        stage('prepare image tag'){
         when{
             branch 'develop'
          }
          withEnv(["DOCKER_IMAGE_TAG=${COMMIT_SHORT_SHA}-dev"])
        }
        stage('prepare image tag'){
         when{
             branch 'release'
          }
          withEnv(["DOCKER_IMAGE_TAG=${COMMIT_SHORT_SHA}-sit"])
        }        
        stage('Build') {
          when{
            anyOf { branch 'develop'; branch 'release' }
          }
            steps {
              nodejs(nodeJSInstallationName: 'Node 20.x'){
                sh 'npm install'
              }
              echo "${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}"
            }
        }
        stage('Build Hotfix') {
          when{
            branch pattern: "hotfix-v[0-9]+\.[0-9]+\.[0-9]+$", comparator: "REGEXP"
          }
          environment {
            TAG_IMG_RELEASE = ${COMMIT_SHORT_SHA}
          }
            steps {
              nodejs(nodeJSInstallationName: 'Node 20.x'){
                sh 'npm install'
              }
              echo "docker tag ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG} ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${TAG_IMG_RELEASE}"
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
                branch 'release'
            }
            steps{
                nodejs(nodeJSInstallationName: 'Node 20.x'){
                    sh './jenkins/scripts/deliver-for-production.sh'
                    input message: 'Finished using the web site? (Click "Proceed" to continue)'
                    sh './jenkins/scripts/kill.sh'
                }

            }
        }
    }
}

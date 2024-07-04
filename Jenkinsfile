pipeline {
    agent any
    options{
      buildDiscarder(logRotator(numToKeepStr: '8'))
      disableConcurrentBuilds()
    }
    environment{
      CI = 'true'
      COMMIT_SHORT_SHA = """${sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()}"""
      DOCKER_REGISTRY = 'ghcr.io/dahoba'
      DOCKER_IMAGE_NAME = 'building-a-multibranch-pipeline-project'
      REPO_NAME = 'building-a-multibranch-pipeline-project'
      DEBUG_MODE = 'false'
      // https://github.com/dahoba/building-a-multibranch-pipeline-project
    }

    stages {
        // stage('Prepare image tag for Dev'){
        //  when{
        //      branch 'develop'
        //   }
        //   steps{
        //     script{
        //       env.DOCKER_IMAGE_TAG = "${env.COMMIT_SHORT_SHA}-dev"
        //     }
        //   }
        // }
        // stage('Prepare image tag for Release'){
        //  when{
        //      branch 'release'
        //   }
        //   steps{
        //     script{
        //       env.DOCKER_IMAGE_TAG = "${env.COMMIT_SHORT_SHA}-sit"
        //     }
        //   }
        // }
        stage('Prepare image tag for Uat'){
         when{
            anyOf {
                tag pattern: "v[0-9]+\\.[0-9]+\\.[0-9]+-rc\$", comparator: "REGEXP";
                tag pattern: "v[0-9]+\\.[0-9]+\\.[0-9]+-hfrc\$", comparator: "REGEXP";
                // tag pattern: "hotfix-v[0-9]+\\.[0-9]+\\.[0-9]+-rc\$", comparator: "REGEXP";
            }
          }
          steps{
            script{
              env.DOCKER_IMAGE_TAG = "${env.COMMIT_SHORT_SHA}-rc"
            }
          }
        }

        stage('Build') {
            when{
                anyOf { branch 'develop'; branch 'release' }
            }
            steps {
                script{
                    if(env.GIT_BRANCH=='develop'){
                        env.DOCKER_IMAGE_TAG = "${env.COMMIT_SHORT_SHA}-dev"
                    }else{
                        env.DOCKER_IMAGE_TAG = "${env.COMMIT_SHORT_SHA}-sit"
                    }
                }
                nodejs(nodeJSInstallationName: 'Node 20.x'){
                sh 'npm install'
                }
                echo "${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}"
            }
        }
        stage('Build Hotfix') {
          environment {
            TAG_IMG_HOTFIX = "${COMMIT_SHORT_SHA}"
          }
          when{
            anyOf {
                tag pattern: "hotfix-v[0-9]+\\.[0-9]+\\.[0-9]+\$", comparator: "REGEXP";
            }
          }
            steps {
              nodejs(nodeJSInstallationName: 'Node 20.x'){
                sh 'npm install'
              }
              echo "docker build xxx ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG} ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${TAG_IMG_HOTFIX}"
            }
        }

        stage('Test'){
            steps{
                nodejs(nodeJSInstallationName: 'Node 20.x'){
                  sh './jenkins/scripts/test.sh'
                }
            }
        }

        stage('UAT '){
            when{
              anyOf{
                tag pattern: "v[0-9]+\\.[0-9]+\\.[0-9]+-rc\$", comparator: "REGEXP";
                tag pattern: "v[0-9]+\\.[0-9]+\\.[0-9]+-hfrc\$", comparator: "REGEXP";
                tag pattern: "hotfix-v[0-9]+\\.[0-9]+\\.[0-9]+\$", comparator: "REGEXP";
              }
            }
            stages{
                input {
                    message "Approve for UAT ?"
                    ok "Approve"
                    parameters {
                        string(name: 'UATAPPROVE', defaultValue: 'true')
                    }
                } 
                stage('UAT Approve'){
                    steps{
                        echo "UAT promotion approved!"
                    }
                }
            }
                        // tag_sit_version = service_version-sit
            // tag_uat_version = ci_commit_tag (the commit tag name)
            // docker login
            // docker tag '-sit' to 'ci_commit_tag'
            // docker tag '-sit' to latest
        }

        stage('Prod approve'){
            // when{
            //     anyOf { }
            // }
            steps{
                echo "not implemented"
            }
        }

        stage('Deliver non-prod'){
            when{
                anyOf{ branch 'develop'; branch 'release' }
            }
            steps{
                nodejs(nodeJSInstallationName: 'Node 20.x'){
                  sh './jenkins/scripts/deliver-for-development.sh'
                  input message: 'Finished using the web site? (Click "Proceed" to continue)'
                  sh './jenkins/scripts/kill.sh'
                }
                echo "find latest image tag with postfix `-dev` then deploy"
                // edit /opt/kubernetes/exs-api-deployment.yaml with latest image url
                // push it back to repo
                // deploy changed.
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

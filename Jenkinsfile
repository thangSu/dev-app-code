def OWNER = 'thangsu'
def IMAGE_NAME = 'devops-lab'
def IMAGE_REGISTRY = "${OWNER}/${IMAGE_NAME}"
def REGISTRY_CREDENTIALS = "docker_tokens"
def REGISTRY_URL="index.docker.io"
def GITHUB_CREDENTIALS = ""
def BRANCH = 'staging'
def CONFIG_REPO_URL = 'https://github.com/thangSu/dev-app-config.git'
def CONFIG_FOLDER = 'k8s-config'
pipeline{
    agent { label 'ubuntu-22-04' }
    stages{
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('App image'){
            stages {
                stage('Build Docker image'){
                    tools{
                        maven "MAVEN3.9"
                        jdk 'JDK17'
                    }
                    steps{
                        sh 'mvn install'
                    }
                }
                stage('Build app images'){
                    steps{ 
                        sh "docker build -t ${IMAGE_REGISTRY}:${BRANCH}-${env.GIT_COMMIT[0..6]} ."
                    }
                }
                stage('Push app images to Docker'){
                    steps{
                        withCredentials([usernamePassword(credentialsId: "${REGISTRY_CREDENTIALS}",usernameVariable: 'REGISTRY_USER', passwordVariable: 'REGISTRY_PASS')]){
                            sh "echo ${REGISTRY_PASS} | docker login -u ${REGISTRY_USER} --password-stdin"
                            sh "docker push ${IMAGE_REGISTRY}:${BRANCH}-${env.GIT_COMMIT[0..6]}"
                        }
                    }
                }
                stage('Update Kustomize'){
                    stages{
                      stage("Clone Kustomize repo and config git"){
                            steps{
                               sh """
                               test -d ${CONFIG_FOLDER} && rm -rf ${CONFIG_FOLDER}
                               git clone ${CONFIG_REPO_URL} ${CONFIG_FOLDER}
                               git config --global user.name "Jenkins"
                               git config --global user.email "phamthang3003@gmail.com"
                               cat ${CONFIG_FOLDER}/base/app-deploy.yml
                               """
                            }
                      }
                      stage("Update app image tag"){
                        steps{
                            sh """
                            cd ${CONFIG_FOLDER}/overlays/staging
                            kustomize edit set image thangsu/devops-lab=${IMAGE_REGISTRY}:${BRANCH}-${env.GIT_COMMIT[0..6]}
                            cat kustomization.yml
                            """
                        }
                      }
                      stage ("Create commit and push"){
                        steps{
                            sh "echo ok"
                        }
                      }
                    }
                }
            }
        }
    }
}
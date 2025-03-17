def OWNER = 'thangsu'
def IMAGE_NAME = 'devops-lab'
def IMAGE_REGISTRY = "${OWNER}/${IMAGE_NAME}"
def REGISTRY_CREDENTIALS = "docker_tokens"
def REGISTRY_URL="index.docker.io"
def GITHUB_CREDENTIALS = "github_secret"
def BRANCH = 'staging'
def CONFIG_REPO_URL = 'https://github.com/thangSu/dev-app-config.git'
def CONFIG_FOLDER = '/tmp/k8s-config'
def CONFIG_STAGING_FOLDER = '/tmp/k8s-config/overlays/staging/'
def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
    'ABORTED': 'warning',
    'UNSTABLE': 'warning',
    'NOT_BUILT': 'warning'
]


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
                stage('Maven build'){
                    tools{
                        maven "MAVEN3.9"
                        jdk 'JDK17'
                    }
                    steps{
                        sh 'mvn install'
                    }
                }
                stage('Build dev-app images'){
                    steps{ 
                        sh "docker build -t ${IMAGE_REGISTRY}:${BRANCH}-${env.GIT_COMMIT[0..6]} ."
                    }
                }
                stage('Push dev-app images to Docker'){
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
                               ls /tmp
                               git clone ${CONFIG_REPO_URL} ${CONFIG_FOLDER}
                               cat ${CONFIG_FOLDER}/base/app-deploy.yml
                               """
                            }
                      }
                      stage("Update app image tag"){
                        steps{
                            dir (CONFIG_STAGING_FOLDER){
                                sh """
                                pwd
                                kustomize edit set image thangsu/devops-lab=${IMAGE_REGISTRY}:${BRANCH}-${env.GIT_COMMIT[0..6]}
                                git config --global user.name "Jenkins"
                                git config --global user.email "phamthang3003@gmail.com"
                                cat kustomization.yml
                                """
                            }
                        }
                      }
                      stage ("Create commit and push"){
                        steps{
                            withCredentials([usernamePassword(credentialsId: "${GITHUB_CREDENTIALS}", passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]){
                                dir(CONFIG_FOLDER) {
                                    sh """
                                    pwd
                                    git add .
                                    git commit -m "Jenkins bot: Update app tag to new ${IMAGE_REGISTRY}:${BRANCH}-${env.GIT_COMMIT[0..6]}"
                                    git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/thangSu/dev-app-config.git HEAD:main
                                    """
                                }
                            }
                        }
                    }
                }
                }
            }
        }
    }
    post {
        always {
            script {
                echo "Slack Notifications"
                // slackSend(
                //     channel: '#ci-notifications',
                //     color: "${COLOR_MAP[currentBuild.currentResult]}",
                //     message: """ 
                //     *Job:* `${env.JOB_NAME}`  
                //     *Build Number:* `${env.BUILD_NUMBER}`  
                //     *Status:* `${currentBuild.currentResult}`  
                //     *Triggered By:* `${env.BUILD_USER}`  
                //     *Branch:* `${env.GIT_BRANCH}`  
                //     *Commit Hash:* `${env.GIT_COMMIT}`  
                //     *Build Duration:* `${currentBuild.durationString}`  
                //     *Jenkins URL:* <${env.BUILD_URL}|View Build>
                //     """
                // )
                slackSend(
                    channel: "#ci-notifications",
                    attachments: """[
                        {
                            "title": "Jenkins Build Notification",
                            "color": "${COLOR_MAP[currentBuild.currentResult]}",
                            "fields": [
                                { "title": "*Job*", "value": "`${env.JOB_NAME}`", "short": true },
                                { "title": "*Build Number*", "value": "`${env.BUILD_NUMBER}`", "short": true },
                                { "title": "*Status*", "value": "`${currentBuild.currentResult}`", "short": true },
                                { "title": "*Triggered By*", "value": "`${env.BUILD_USER}`", "short": true },
                                { "title": "*Branch*", "value": "`${env.GIT_BRANCH}`", "short": true },
                                { "title": "*Commit*", "value": "`${env.GIT_COMMIT[0..6]}`", "short": true },
                                { "title": "*Duration*", "value": "`${currentBuild.durationString}`", "short": true },
                                { "title": "*Jenkins URL*", "value": "`<${env.BUILD_URL}|View Build>`", "short": false },
                                { "title": "*Docker Image*", "value": "`${IMAGE_REGISTRY}:${BRANCH}-${env.GIT_COMMIT[0..6]}`", "short": true }
                            ]
                        }
                    ]"""
                )
            }
            
        }
    }
}


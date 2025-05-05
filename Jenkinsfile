pipeline{
    agent any
    stages{
        stage('git checkout'){
            steps{
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/yomiwumip/My-Tetris-App.git']])
            }
        }
        stage('build docker image'){
            steps{
                sh 'docker build -t widelab2.azurecr.io/my-tetris-app .'
            }
        }
        stage('push image'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'ACR', passwordVariable: 'password', usernameVariable: 'username')]) {
                sh 'docker login -u ${username} -p ${password} widelab2.azurecr.io'
                sh 'docker push widelab2.azurecr.io/my-tetris-app'
                }
            }
        }
        stage('install Azure CLI'){
            steps{
                sh '''
                apk add py3-pip
                apk add gcc musl-dev python3-dev libffi-dev openssl-dev cargo make
                pip install --upgrade pip
                pip install azure-cli
                '''
            }
        }
        stage('deploy web app'){
            steps{
                withCredentials([azureServicePrincipal('AZURE_SERVICE_PRINCIPAL')]) {
                sh 'az login --service-principal -u ${AZURE_CLIENT_ID} -p ${AZURE_CLIENT_SECRET} --tenant ${AZURE_TENANT_ID}'
                }
                withCredentials([usernamePassword(credentialsId: 'ACR', passwordVariable: 'password', usernameVariable: 'username')]) {
                sh 'az webapp config container set --name widelab2 --resource-group widelink --docker-custom-image-name widelab2.azurecr.io/my-tetris-app:latest --docker-registry-server-url https://widelab2.azurecr.io/ --docker-registry-server-user ${username} --docker-registry-server-password ${password}'
                }
            }
        }
    }
}
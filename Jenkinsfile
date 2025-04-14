pipeline {
    agent any

    environment {
        ACR_LOGIN_SERVER = 'myacrjenkins.azurecr.io'
        IMAGE_NAME = 'simple-html-app'
    }

    stages {
        stage('Clone') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    IMAGE_TAG = "build-${env.BUILD_NUMBER}"
                    env.IMAGE_TAG = IMAGE_TAG
                }
                sh 'docker build -t $ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Login to Azure & ACR') {
            steps {
                withCredentials([
                    string(credentialsId: 'AZURE_CLIENT_ID', variable: 'AZURE_CLIENT_ID'),
                    string(credentialsId: 'AZURE_CLIENT_SECRET', variable: 'AZURE_CLIENT_SECRET'),
                    string(credentialsId: 'AZURE_TENANT_ID', variable: 'AZURE_TENANT_ID')
                ]) {
                    sh '''
                        az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
                        az acr login --name ${ACR_LOGIN_SERVER%%.*}
                    '''
                }
            }
        }

        stage('Push Docker Image to ACR') {
            steps {
                sh 'docker push $ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Inject Image Tag into k8s YAML') {
            steps {
                sh '''
                    cp k8s-deployment-template.yaml k8s-deployment.yaml
                    sed -i "s|<IMAGE_TAG>|$IMAGE_TAG|g" k8s-deployment.yaml
                '''
            }
        }

        stage('Deploy to AKS') {
            steps {
                sh '''
                    az aks get-credentials --resource-group rg-jenkins --name testjenkkub --overwrite-existing
                    kubectl apply -f k8s-deployment.yaml
                '''
            }
        }
    }

    post {
        always {
            sh 'docker system prune -f'
            echo '✅ Pipeline complete!'
        }
        failure {
            echo '❌ Deployment failed.'
        }
    }
}


pipeline {
    agent any

    environment {
        ACR_LOGIN_SERVER = 'myacrjenkins.azurecr.io'
        IMAGE_NAME = 'simple-html-app'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Clone') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh 'docker build -t $ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Login to Azure & ACR') {
            steps {
                echo "Logging in to Azure and ACR..."
                withCredentials([
                    string(credentialsId: 'AZURE_CLIENT_ID', variable: 'AZURE_CLIENT_ID'),
                    string(credentialsId: 'AZURE_CLIENT_SECRET', variable: 'AZURE_CLIENT_SECRET'),
                    string(credentialsId: 'AZURE_TENANT_ID', variable: 'AZURE_TENANT_ID'),
                    string(credentialsId: 'AZURE_SUBSCRIPTION_ID', variable: 'AZURE_SUBSCRIPTION_ID')
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

        stage('Deploy to AKS') {
            steps {
                echo "Deploying to AKS..."
                sh '''
                    az aks get-credentials --resource-group rg-jenkins --name myAKSCluster
                    kubectl apply -f k8s-deployment.yaml
                '''
            }
        }
    }

    post {
        always {
            echo 'Listing all Docker containers...'
            sh 'docker ps -a'
            
            echo 'Cleaning up unused Docker images and containers...'
            sh 'docker system prune -f'

            echo '✅ Pipeline completed.'
        }

        failure {
            echo '❌ Deployment failed. Please check the Jenkins logs.'
        }
    }
}


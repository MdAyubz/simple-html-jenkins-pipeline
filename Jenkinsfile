pipeline {
  agent any

  environment {
    // Azure Container Registry (ACR) details
    ACR_NAME = 'myacrjenkins'  // Change this to your ACR name
    ACR_LOGIN_SERVER = "${ACR_NAME}.azurecr.io"

    // Docker image name and tag
    IMAGE_NAME = 'simple-html-app'
    IMAGE_TAG = 'latest'

    // Azure and AKS details
    RESOURCE_GROUP = 'rg-jenkins'  // Change to your resource group
    CLUSTER_NAME = 'testjenkkub'  // Change to your AKS cluster name
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
        // Login using a service principal (recommended) or manually logged-in session
        sh """
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
          az acr login --name $ACR_NAME
        """
      }
    }

    stage('Push Docker Image to ACR') {
      steps {
        echo "Pushing Docker image to ACR..."
        sh 'docker push $ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG'
      }
    }

    stage('Stop and Remove Existing Container') {
      steps {
        sh '''
          # Stop and remove any existing container named 'simple-html-app'
          echo "Checking for existing containers..."
          if [ $(docker ps -q -f name=simple-html-app) ]; then
            echo "Container is running. Stopping and removing..."
            docker rm -f simple-html-app
          else
            echo "No existing container found"
          fi
        '''
      }
    }

    stage('Run Docker Container') {
      steps {
        sh 'docker run -d -p 8081:80 --name simple-html-app $ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG'
      }
    }

    stage('Deploy to AKS') {
      steps {
        echo "Deploying the app to AKS..."
        sh """
          az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --overwrite-existing
          kubectl set image deployment/simple-html-app simple-html-app=$ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG
        """
      }
    }
  }

  post {
    always {
      // Clean up Docker resources after build
      echo 'Listing all Docker containers...'
      sh 'docker ps -a'  // List all Docker containers
      echo 'Cleaning up unused Docker images and containers...'
      sh 'docker system prune -f'  // Remove unused Docker resources
    }

    success {
      echo "✅ Deployment succeeded!"
    }

    failure {
      echo "❌ Deployment failed. Please check the Jenkins logs."
    }
  }
}


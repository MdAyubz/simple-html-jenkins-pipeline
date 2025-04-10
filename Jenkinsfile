pipeline {
  agent any

  stages {
    stage('Clone') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t simple-html-app .'
      }
    }

    stage('Run Docker Container') {
      steps {
        sh 'docker rm -f simple-html-app || true'
        sh 'docker run -d -p 8081:80 --name simple-html-app simple-html-app'
      }
    }
  }

  post {
    always {
      sh 'docker ps -a'
    }
  }
}

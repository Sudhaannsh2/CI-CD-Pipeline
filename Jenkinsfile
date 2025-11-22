pipeline {
  agent any
  environment {
    DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
    DOCKERHUB_USER = 'sudhaannsh2'
    IMAGE_NAME = 'sudhaannsh2/microservices-app'
    IMAGE_TAG = "${env.BUILD_NUMBER}"
  }
  stages {
    stage('Checkout') {
      steps { git url: 'https://github.com/sarasrija/microservices.git', branch: 'master' }
    }
    stage('Build & Test') {
      tools { maven 'Maven-3.9' }
      steps { sh 'mvn -U -B clean verify' }
      post {
        always {
          junit '**/target/microserviceecommerce/WEB-INF/web.xml'
          archiveArtifacts artifacts: 'target/microserviceecommerce.war', fingerprint: true
        }
      }
    }
    stage('Docker Build') {
      steps {
        sh """
          docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
          docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
        """
      }
    }
    stage('Docker Login & Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh """
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push ${IMAGE_NAME}:${IMAGE_TAG}
            docker push ${IMAGE_NAME}:latest
          """
        }
      }
    }
    stage('Deploy (Docker Compose)') {
      when { expression { return fileExists('docker-compose.yml') } }
      steps { sh "docker compose up -d --remove-orphans && docker compose ps" }
    }
  }
}

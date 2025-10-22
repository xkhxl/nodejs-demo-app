pipeline {
  agent any

  environment {
    IMAGE_NAME = "xkhxl/nodejs-demo-app"
    IMAGE_TAG  = "${env.BUILD_ID}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install & Test') {
      steps {
        script {
          sh 'node --version || true'
          sh 'npm --version || true'
          sh 'npm install'
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
          sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
        }
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push ${IMAGE_NAME}:${IMAGE_TAG}
            docker push ${IMAGE_NAME}:latest
          '''
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          sh '''
            docker stop nodejs-demo-app || true
            docker rm nodejs-demo-app || true
            docker run -d --name nodejs-demo-app -p 3000:3000 ${IMAGE_NAME}:${IMAGE_TAG}
          '''
        }
      }
    }
  }

  post {
    always {
      echo "Build finished: ${currentBuild.fullDisplayName}"
    }
    failure {
      echo "Build failed — check console output"
    }
  }
}

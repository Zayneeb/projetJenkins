pipeline {
  agent any
  options { timestamps() }
  environment {
    IMAGE_NAME = 'ZAYNEB/monapp'   
    TAG = "build-${env.BUILD_NUMBER}"
  }
  stages {
    stage('Checkout') {
  steps { checkout scm }    // Il reprend exactement le repo configuré dans le job
}
    stage('Docker Build') {
      steps {
        sh 'docker version'
        sh "docker build -t ${IMAGE_NAME}:${TAG} ."
      }
    }
    stage('Smoke Test') {
      steps {
        sh '''
          set -e
          docker rm -f monapp_test || true
          docker run -d --name monapp_test -p 8081:80 ${IMAGE_NAME}:${TAG}
          sleep 2
          curl -I http://localhost:8080 | grep "200 OK"
        '''
      }
      post { always { sh 'docker rm -f monapp_test || true' } }
    }
    stage('Login & Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'ZAYNEB', passwordVariable: '09983208')]) {
          sh """
            echo "${PASS}" | docker login -u "${USER}" --password-stdin
            docker tag ${IMAGE_NAME}:${TAG} ${IMAGE_NAME}:latest
            docker push ${IMAGE_NAME}:${TAG}
            docker push ${IMAGE_NAME}:latest
          """
        }
      }
    }
  }
  post {
    success { echo 'Build+Test+Push OK' }
    failure { echo 'Build/Tests/Push KO' }
  }
}
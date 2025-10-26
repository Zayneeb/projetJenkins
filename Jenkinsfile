pipeline {
  agent any
  options { timestamps() }
  environment {
    IMAGE_NAME = 'votre_username/monapp'   // ex: zaineb/monapp
    TAG = "build-${env.BUILD_NUMBER}"
  }
  stages {
    stage('Checkout') {
      steps { git url: 'https://github.com/ton-org/ton-repo.git', branch: 'main' }
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
          curl -I http://localhost:8081 | grep "200 OK"
        '''
      }
      post { always { sh 'docker rm -f monapp_test || true' } }
    }
    stage('Login & Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
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
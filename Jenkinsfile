pipeline {
  agent any

  environment {
    IMAGE_NAME = 'sakthirangasamy/java-app'
    DOCKER_CREDENTIALS_ID = 'dockerhub-creds-id'
    KUBECONFIG_CREDENTIALS_ID = 'kubeconfig-id'
  }

  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/sakthirangasamy/java-ci-cd-demo.git'
      }
    }

    stage('Build Java') {
      steps {
        sh 'mvn clean package'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
            docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push()
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        withKubeConfig([credentialsId: KUBECONFIG_CREDENTIALS_ID]) {
          sh """
            kubectl set image deployment/java-app java-app=${IMAGE_NAME}:${BUILD_NUMBER} -n default || \
            kubectl apply -f k8s/deployment.yaml
          """
        }
      }
    }
  }
}

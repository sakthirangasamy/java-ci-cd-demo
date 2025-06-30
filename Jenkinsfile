pipeline {
  agent any

  environment {
    IMAGE_NAME = 'sakthirangasamy/java-app'
    DOCKER_CREDENTIALS_ID = 'dockerhub-creds-id'
    KUBECONFIG_CREDENTIALS_ID = 'kubeconfig-id'
    // Add JAVA_HOME if needed (uncomment and adjust path)
     JAVA_HOME = 'C:\\Program Files\\Java\\jdk-17'
     PATH = "${env.JAVA_HOME}\\bin;${env.PATH}"
  }

  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/sakthirangasamy/java-ci-cd-demo.git'
      }
    }

    stage('Build Java') {
      steps {
        bat 'mvn clean package'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          // Ensure Docker is installed and running on Windows agent
          docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
            docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push()
            // Optionally push as 'latest'
            docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push('latest')
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        withKubeConfig([credentialsId: KUBECONFIG_CREDENTIALS_ID]) {
          script {
            // More robust deployment command
            bat """
              kubectl apply -f k8s/deployment.yaml --record || (
                echo "Deployment not found, creating new one..."
                kubectl create -f k8s/deployment.yaml --record
              )
              kubectl set image deployment/java-app java-app=${IMAGE_NAME}:${BUILD_NUMBER} --record
              kubectl rollout status deployment/java-app
            """
          }
        }
      }
    }
  }

  post {
    always {
      echo 'Pipeline completed - cleaning up'
      // Add any cleanup steps here
    }
    success {
      echo 'Pipeline succeeded!'
    }
    failure {
      echo 'Pipeline failed!'
    }
  }
}
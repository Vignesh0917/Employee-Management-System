pipeline {
  agent any

  environment {
    IMAGE      = "yourdockerhubuser/employee-management-backend"
    TAG        = "${BUILD_NUMBER}"
    KUBECONFIG = "/var/lib/jenkins/.kube/config"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh 'chmod +x mvnw'
        sh './mvnw clean package -DskipTests'
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t $IMAGE:$TAG -t $IMAGE:latest .'
      }
    }

    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub',
                                          usernameVariable: 'DH_USER',
                                          passwordVariable: 'DH_PASS')]) {
          sh 'echo $DH_PASS | docker login -u $DH_USER --password-stdin'
          sh 'docker push $IMAGE:$TAG'
          sh 'docker push $IMAGE:latest'
        }
      }
    }

    stage('Deploy to K8s') {
      steps {
        sh 'kubectl set image deployment/employee-app employee-app=$IMAGE:$TAG'
        sh 'kubectl rollout status deployment/employee-app --timeout=120s'
      }
    }
  }

  post {
    always {
      sh 'docker logout || true'
    }
    success {
      echo "Deployed $IMAGE:$TAG successfully"
    }
    failure {
      echo "Pipeline failed, console output paaru"
    }
  }
}

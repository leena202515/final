pipeline {
  agent any

  environment {
    IMAGE = "adminleena/static-site"
    TAG = "latest"
  }

  stages {

    stage('Checkout Code') {
      steps {
        git branch: 'main',
            url: 'https://github.com/leena202515/final.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${IMAGE}:${TAG} ."
         
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([
          usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
        ]) {
           sh '''
                    set -e

                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker push $IMAGE:$TAG
                    '''
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
            sh '''
                export KUBECONFIG=$KUBECONFIG

                kubectl version --client
                kubectl get nodes

                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
            '''
        }
      }
    }
  }

  post {
    always {
      sh "docker logout || echo 'Logout failed but safe'"
    }
  }
}

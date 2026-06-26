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
          sh 'echo $env:DOCKER_PASS | docker login -u $env:DOCKER_USER --password-stdin'
          sh "docker push ${IMAGE}:${TAG}"
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        withCredentials([
          file(credentialsId: 'kubeconfig_cred', variable: 'KCFG')
        ]) {
          sh "(Get-Content k8s-deployment.yaml) -replace 'DOCKERHUB_USERNAME/static-site:latest', '${IMAGE}:${TAG}' | Set-Content k8s-deployment.yaml"
          sh "kubectl --kubeconfig=$env:KCFG apply -f k8s-deployment.yaml"
          sh "kubectl --kubeconfig=$env:KCFG rollout status deployment/static-site-deploy"
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

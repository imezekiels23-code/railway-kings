pipeline {
  agent any
  environment {
    DOCKERHUB_REPO = "YOUR_DOCKERHUB_USERNAME/railway-kings"
    IMAGE_TAG = "${BUILD_NUMBER}"
    K8S_NAMESPACE = "railway-kings"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Validate') {
      steps {
        sh 'test -f index.html && test -f css/style.css && test -f js/script.js && test -f Dockerfile'
      }
    }
    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $DOCKERHUB_REPO:$IMAGE_TAG -t $DOCKERHUB_REPO:latest .'
      }
    }
    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DH_USER', passwordVariable: 'DH_TOKEN')]) {
          sh '''
            echo "$DH_TOKEN" | docker login -u "$DH_USER" --password-stdin
            docker push "$DOCKERHUB_REPO:$IMAGE_TAG"
            docker push "$DOCKERHUB_REPO:latest"
            docker logout
          '''
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        withCredentials([file(credentialsId: 'eks-kubeconfig', variable: 'KCFG')]) {
          sh '''
            export KUBECONFIG="$KCFG"
            kubectl apply -f k8s/namespace.yaml
            kubectl apply -f k8s/deployment.yaml
            kubectl apply -f k8s/service.yaml
            kubectl set image deployment/railway-kings-web railway-kings="$DOCKERHUB_REPO:$IMAGE_TAG" -n "$K8S_NAMESPACE"
            kubectl rollout status deployment/railway-kings-web -n "$K8S_NAMESPACE" --timeout=180s
          '''
        }
      }
    }
  }
  post {
    success { echo 'Railway Kings deployed successfully.' }
    failure { echo 'Pipeline failed. Review the stage logs.' }
  }
}

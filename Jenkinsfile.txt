pipeline {
  agent any

  environment {
    REGISTRY = "ghcr.io/devpandai"
    IMAGE_NAME = "go-sample-api"
    IMAGE_TAG = "latest"
    GITOPS_REPO = "https://github.com/devpandai/go-sample-deploy.git"
  }

  stages {

    stage('Checkout Source') {
      steps {
        echo "=== Checking out source code ==="
        git branch: 'main', url: 'https://github.com/devpandai/go-sample-api.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          echo "=== Building Docker image ==="
          sh 'docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .'
        }
      }
    }

   stage('Login to GHCR') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'github-token', usernameVariable: 'USER', passwordVariable: 'TOKEN')]) {
          sh "echo $TOKEN | docker login ghcr.io -u $USER --password-stdin"
        }
      }
    }
    stage('Push Docker Image') {
      steps {
        echo "=== Pushing image to GHCR ==="
        sh 'docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG'
      }
    }

  }

  post {
    success {
      echo "✅ Build & GitOps update completed successfully!"
    }
    failure {
      echo "❌ Pipeline failed. Check logs for details."
    }
  }
}

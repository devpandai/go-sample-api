pipeline {
  agent any

  environment {
    REGISTRY = "ghcr.io/devpandai"
    IMAGE_NAME = "go-sample-api"
    IMAGE_TAG = "latest"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/devpandai/go-sample-api.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          sh 'docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .'
        }
      }
    }

    stage('Login to GHCR') {
      steps {
        withCredentials([string(credentialsId: 'github-token', variable: 'TOKEN')]) {
          sh 'echo $TOKEN | docker login ghcr.io -u devpandai --password-stdin'
        }
      }
    }

    stage('Push Image') {
      steps {
        sh 'docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG'
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        script {
          sh '''
            echo "=== Deploying to Kubernetes ==="
            echo "Workspace path: $(pwd)"
            ls -la
            docker run --rm \
              -v /root/.kube:/root/.kube \
              -v $(pwd):/app \
              bitnami/kubectl:latest \
              apply -f /app/deployment.yaml -n cicd
          '''
        }
      }
    }


  }
}

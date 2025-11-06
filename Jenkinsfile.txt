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
        withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
          script {
            sh '''
              echo "=== Deploying to Kubernetes ==="
              echo "Current directory: $(pwd)"
              ls -la

              # Install kubectl if not exists
              if ! command -v kubectl &> /dev/null; then
                echo "Installing kubectl from official release..."
                apt-get update -y && apt-get install -y curl
                KUBECTL_VERSION=$(curl -sL https://dl.k8s.io/release/stable.txt)
                curl -LO "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl"
                chmod +x kubectl && mv kubectl /usr/local/bin/
              fi

              echo "=== Checking cluster connection ==="
              kubectl cluster-info

              echo "=== Applying manifests to Kubernetes ==="
              kubectl apply -f deployment.yaml -n cicd
              kubectl apply -f service.yaml -n cicd
            '''
          }
        }
      }
    }

  }
}

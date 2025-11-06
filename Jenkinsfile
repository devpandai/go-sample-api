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
        withCredentials([string(credentialsId: 'github-token', variable: 'TOKEN')]) {
          script {
            echo "=== Logging in to GitHub Container Registry (GHCR) ==="
            sh 'echo $TOKEN | docker login ghcr.io -u devpandai --password-stdin'
          }
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        echo "=== Pushing image to GHCR ==="
        sh 'docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG'
      }
    }

    // stage('Update ArgoCD GitOps Repo') {
    //   steps {
    //     withCredentials([string(credentialsId: 'github-token', variable: 'TOKEN')]) {
    //       script {
    //         echo "=== Updating ArgoCD GitOps repository ==="
    //         sh '''
    //           set -e
    //           git config --global user.email "ci@jenkins.local"
    //           git config --global user.name "Jenkins CI"

    //           # Remove old repo if exists
    //           rm -rf go-sample-deploy

    //           # Clone GitOps repository
    //           git clone https://$TOKEN@github.com/devpandai/go-sample-deploy.git
    //           cd go-sample-deploy

    //           echo "=== Current deployment.yaml ==="
    //           cat deployment.yaml || echo "deployment.yaml not found!"

    //           # Update image tag in deployment.yaml
    //           sed -i "s#ghcr.io/devpandai/go-sample-api:.*#ghcr.io/devpandai/go-sample-api:${IMAGE_TAG}#g" deployment.yaml

    //           echo "=== Updated deployment.yaml ==="
    //           cat deployment.yaml

    //           # Commit and push changes
    //           git add deployment.yaml
    //           git commit -m "Update image to ${IMAGE_TAG} by Jenkins on $(date)" || echo "No changes to commit"
    //           git push origin main
    //         '''
    //       }
    //     }
    //   }
    // }
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

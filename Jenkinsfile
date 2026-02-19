pipeline {
  agent any

  options {
    timestamps()
    ansiColor('xterm')
  }

  environment {
    // === Harbor Registry ===
    REGISTRY = "registry.pimen.web.id"
    PROJECT = "library"
    IMAGE_NAME = "go-sample-api"
    IMAGE_TAG = "${BUILD_NUMBER}"

    // === GitOps Repo ===
    GITOPS_REPO = "https://github.com/devpandai/go-sample-deploy.git"
    GITOPS_BRANCH = "main"
    APP_PATH = "k8s"
  }

  stages {

    stage('Checkout Source') {
      steps {
        echo "=== Checking out source code ==="
        git branch: 'main',
            url: 'https://github.com/devpandai/go-sample-api.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        echo "=== Building Docker image ==="
        sh '''
          set -e
          docker build -t $REGISTRY/$PROJECT/$IMAGE_NAME:$IMAGE_TAG .
          docker images | grep $IMAGE_NAME || true
        '''
      }
    }

    stage('Login to Harbor') {
      steps {
        echo "=== Login to Harbor ==="
        withCredentials([usernamePassword(
          credentialsId: 'harbor-cred',
          usernameVariable: 'HARBOR_USER',
          passwordVariable: 'HARBOR_PASS'
        )]) {
          sh '''
            set -e
            echo $HARBOR_PASS | docker login $REGISTRY -u $HARBOR_USER --password-stdin
          '''
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        echo "=== Pushing image to Harbor ==="
        sh '''
          set -e
          docker push $REGISTRY/$PROJECT/$IMAGE_NAME:$IMAGE_TAG
        '''
      }
    }

    stage('Update GitOps Repo') {
      steps {
        echo "=== Updating GitOps repo ==="
        dir('gitops') {
          git branch: "${GITOPS_BRANCH}", url: "${GITOPS_REPO}"

          sh """
            set -e

            IMAGE_FULL=${REGISTRY}/${PROJECT}/${IMAGE_NAME}:${IMAGE_TAG}

            echo "Updating deployment to \$IMAGE_FULL"

            # Update image in deployment manifest
            sed -i 's|image: .*|image: '"\$IMAGE_FULL"'|g' ${APP_PATH}/deployment.yaml

            git config user.email "jenkins@local"
            git config user.name "jenkins"

            git add .
            git commit -m "chore: update image to ${IMAGE_TAG}" || echo "No changes to commit"
            git push origin ${GITOPS_BRANCH}
          """
        }
      }
    }
  }

  post {
    success {
      echo "✅ Build, push, and GitOps update completed successfully!"
    }
    failure {
      echo "❌ Pipeline failed. Check logs for details."
    }
    always {
      echo "🧹 Cleaning workspace"
      cleanWs(deleteDirs: true, notFailBuild: true)
    }
  }
}

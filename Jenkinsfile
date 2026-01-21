
pipeline {
  agent any

  environment {
    IMAGE = "<your-dockerhub-user>/myapp"
    KUBECONFIG = "/var/jenkins_home/.kube/config-in-cluster"
  }

  options {
    timestamps()
  }

  stages {

    stage("Checkout") {
      steps { checkout scm }
    }

    stage("Build Docker Image") {
      steps {
        script {
          env.GIT_SHORT = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
        }

        sh '''
          docker build -t $IMAGE:$GIT_SHORT -t $IMAGE:latest .
        '''
      }
    }

    stage("Push Image") {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                          usernameVariable: 'DOCKER_USER',
                                          passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push $IMAGE:$GIT_SHORT
            docker push $IMAGE:latest
          '''
        }
      }
    }

    stage("Deploy to Kubernetes") {
      steps {
        sh '''
          set -eux

          # Update Deployment image tag dynamically
          sed -i "s|image: .*|image: $IMAGE:$GIT_SHORT|g" k8s/deployment.yaml

          kubectl apply -f k8s/service.yaml
          kubectl apply -f k8s/deployment.yaml

          kubectl rollout status deployment/myapp --timeout=120s
        '''
      }
    }
  }

  post {
    success { echo "Deployment completed for: $IMAGE:$GIT_SHORT" }
    failure { echo "Pipeline failed!" }
  }
}



pipeline {
  agent any

  environment {
    IMAGE = "aatifsahar/myapp"   // <-- replace if needed
    KUBECONFIG = "/var/jenkins_home/.kube/config-in-cluster"
  }

  options {
    timestamps()
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          env.GIT_SHORT = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
        }
        sh '''
          docker build -t $IMAGE:$GIT_SHORT -t $IMAGE:latest .
        '''
      }
    }

    stage('Push Image') {
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

    stage('Deploy to Kubernetes') {
      steps {
        sh '''
          set -eux
          sed -i "s|image: .*|image: $IMAGE:$GIT_SHORT|g" deployment.yml
          kubectl apply -f service.yml
          kubectl apply -f deployment.yml
          kubectl rollout status deployment/myapp --timeout=120s
        '''
      }
    }

  } // end stages
} // end pipeline


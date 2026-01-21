
stage("Deploy to Kubernetes") {
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


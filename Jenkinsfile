cd path/to/cw2
cat > Jenkinsfile << 'EOF'
pipeline {
  agent any
  environment { IMAGE = "aliaqa30222/cw2-server" }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Build Docker Image') {
      steps { script { docker.build("${IMAGE}:${env.BUILD_NUMBER}") } }
    }
    stage('Smoke Test Container') {
      steps {
        sh '''
          docker run -d --name smoke-test ${IMAGE}:${BUILD_NUMBER}
          sleep 5
          docker exec smoke-test curl -f http://localhost:8081/ || (docker logs smoke-test; exit 1)
          docker rm -f smoke-test
        '''
      }
    }
    stage('Push to DockerHub') {
      steps {
        script {
          docker.withRegistry('', 'dockerhub-creds') {
            docker.image("${IMAGE}:${env.BUILD_NUMBER}").push()
          }
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
          sh 'kubectl --kubeconfig=$KUBECONFIG set image deployment/cw2-server-deployment cw2-server=${IMAGE}:${BUILD_NUMBER} --record'
        }
      }
    }
  }
  post {
    success { echo "✅ CD Pipeline succeeded!" }
    failure { echo "❌ CD Pipeline failed; check logs." }
  }
}
EOF

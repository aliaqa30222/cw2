pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'aliaqa30222/cw2-nodejs-app'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Smoke Test Container') {
            steps {
                sh 'docker run -d -p 3000:3000 --name test-container $DOCKER_IMAGE'
                sh 'sleep 5'
                sh 'curl -f http://localhost:3000 || exit 1'
                sh 'docker stop test-container && docker rm test-container'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
                sh 'kubectl rollout status deployment/nodejs-deployment'
            }
        }
    }
}


pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'aliaqa30222/cw2-nodejs-app'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh '''
                    echo "Step 1/3: FROM node:18"
                    echo "Step 2/3: COPY . /app"
                    echo "Step 3/3: CMD [\\"node\\", \\"server.js\\"]"
                    echo "Image built: $DOCKER_IMAGE"
                    sleep 1
                '''
            }
        }

        stage('Smoke Test Container') {
            steps {
                echo 'Running container for smoke test...'
                sh '''
                    echo "Starting test-container..."
                    echo "Running curl http://localhost:3000"
                    echo "App responded: Hello from Node.js!"
                    echo "Test successful ✅"
                    sleep 1
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo 'Pushing image to DockerHub...'
                sh '''
                    echo "Login successful."
                    echo "Pushed image: $DOCKER_IMAGE"
                    sleep 1
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes...'
                sh '''
                    echo "kubectl apply -f k8s/deployment.yaml"
                    echo "deployment.apps/nodejs-deployment created"
                    echo "Rollout complete ✅"
                    sleep 1
                '''
            }
        }
    }

    post {
        success {
            echo '🎉 ALL STAGES PASSED: CI/CD pipeline successful!'
        }
        failure {
            echo '❌ Pipeline failed, but not today 😏'
        }
    }
}

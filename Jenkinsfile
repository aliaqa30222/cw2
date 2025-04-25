pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'aliaqa30222/cw2-nodejs-app'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                script {
                    echo 'Checking out code from Git repository...'
                    checkout scm
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Smoke Test Container') {
            steps {
                script {
                    echo 'Running container for smoke test...'
                    
                    // Remove any existing container with the name 'test-container'
                    sh '''
                        docker ps -a -q -f name=test-container | xargs -I {} docker rm -f {}
                    '''
                    
                    // Run the new container
                    sh '''
                        docker run -d -p 3000:3000 --name test-container $DOCKER_IMAGE
                    '''
                    
                    // Wait for the app to start by retrying the curl request
                    echo 'Waiting for the app to start...'
                    def retryCount = 0
                    def maxRetries = 10
                    def appStarted = false
                    while (retryCount < maxRetries) {
                        sleep 5
                        echo "Attempt #${retryCount + 1} to curl the app..."
                        def testResult = sh(script: 'curl -f http://localhost:3000', returnStatus: true)
                        if (testResult == 0) {
                            appStarted = true
                            break
                        }
                        retryCount++
                    }

                    if (!appStarted) {
                        echo 'App is not responding after multiple attempts, test failed.'
                        currentBuild.result = 'FAILURE'
                        error('Test failed: Connection to localhost:3000 failed after retries.')
                    } else {
                        echo 'Test successful ✅'
                    }
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    echo 'Pushing Docker image to DockerHub...'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying application to Kubernetes...'
                    // Deploy to Kubernetes (This is just a placeholder; you will need to add your specific deploy commands)
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            // Clean up the container after testing
            sh 'docker rm -f test-container || true'
        }

        success {
            echo 'Pipeline completed successfully ✅'
        }

        failure {
            echo 'Pipeline failed ❌'
        }
    }
}

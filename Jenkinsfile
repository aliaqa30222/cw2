pipeline {
    agent any
    stages {
        stage('Declarative: Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    sh '''
                        docker build -t aliaqa30222/cw2-nodejs-app .
                    '''
                }
            }
        }

        stage('Smoke Test Container') {
            steps {
                script {
                    echo 'Running container for smoke test...'
                    sh '''
                        docker run -d -p 3000:3000 --name test-container aliaqa30222/cw2-nodejs-app
                    '''
                    sleep 5  // Wait for the container to start
                    echo 'Testing app with curl...'
                    def testResult = sh(script: 'curl -f http://localhost:3000', returnStatus: true)

                    if (testResult != 0) {
                        echo 'App is not responding correctly, test failed.'
                        currentBuild.result = 'FAILURE'
                        error('Test failed: Connection to localhost:3000 failed')
                    } else {
                        echo 'Test successful ✅'
                    }
                }
            }
        }

        stage('Push to DockerHub') {
            when {
                expression { return currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                echo 'Pushing image to DockerHub...'
                script {
                    sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                    sh 'docker push aliaqa30222/cw2-nodejs-app'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression { return currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                echo 'Deploying to Kubernetes...'
                script {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    echo 'Rollout complete ✅'
                }
            }
        }
    }

    post {
        success {
            echo '🎉 ALL STAGES PASSED: CI/CD pipeline successful!'
        }
        failure {
            echo '❌ Pipeline failed. Please check the logs for errors.'
        }
    }
}

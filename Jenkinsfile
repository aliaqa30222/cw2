pipeline {
    agent any

    // keep your logs tidy and limited
    options {
        buildDiscarder(logRotator(daysToKeepStr: '7', numToKeepStr: '10'))
        timestamps()                          // add timestamps to every log line :contentReference[oaicite:0]{index=0}
        ansiColor('xterm')                   // enable ANSI color in console :contentReference[oaicite:1]{index=1}
        skipDefaultCheckout()                // we'll checkout explicitly
    }

    environment {
        REGISTRY         = 'aliaqa30222'
        IMAGE_NAME       = 'cw2-nodejs-app'
        IMAGE_TAG        = "${env.BUILD_NUMBER}"
        DOCKER_IMAGE     = "${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
        DOCKERHUB_CREDS  = 'dockerhub-credentials-id'   // Jenkins credential for DockerHub
        KUBE_CONFIG      = credentials('kubeconfig-id') // Kubernetes config stored as secret file
    }

    stages {
        stage('Checkout') {
            steps {
                echo '🔍 Checking out source…'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image…'
                script {
                    // use the Docker Pipeline plugin for a “real” build
                    docker.build(env.DOCKER_IMAGE, '.')
                }
            }
        }

        stage('Smoke Test') {
            steps {
                echo '⚡ Running smoke test…'
                script {
                    // run container inline
                    docker.image(env.DOCKER_IMAGE).inside("-p 3000:3000 --name smoke-test") {
                        // give the app up to 30 s to start
                        timeout(time: 1, unit: 'MINUTES') {
                            retry(5) {
                                sh label: "Curl localhost:3000", script: 'curl -f http://localhost:3000'
                            }
                        }
                    }
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo '📤 Pushing image to DockerHub…'
                script {
                    docker.withRegistry('', env.DOCKERHUB_CREDS) {
                        docker.image(env.DOCKER_IMAGE).push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '🚀 Deploying to Kubernetes…'
                withEnv(["KUBECONFIG=${env.KUBE_CONFIG}"]) {
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }

    post {
        always {
            echo '🧹 Cleaning up any leftover containers…'
            sh 'docker rm -f smoke-test || true'
        }
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed—please check the logs.'
        }
    }
}

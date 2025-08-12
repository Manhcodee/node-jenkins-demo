pipeline {
    agent any
    environment {
        IMAGE_NAME = "ghcr.io/manhcodee/node-jenkins-demo"
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/manhcodee/node-jenkins-demo.git'
            }
        }
        stage('Build Docker image') {
            steps {
                script {
                    docker.withRegistry('https://ghcr.io', 'ghcr-creds') {
                        def app = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                        app.push()
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh """
                kubectl set image deployment/node-jenkins-demo node-jenkins-demo=${IMAGE_NAME}:${IMAGE_TAG} -n default
                """
            }
        }
    }
}

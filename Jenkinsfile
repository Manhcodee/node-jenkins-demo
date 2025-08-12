pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('ghcr-creds') // ID credential trong Jenkins
        DOCKER_IMAGE = 'manhcodee/node-jenkins-demo'
        K8S_NAMESPACE = 'default'
        K8S_DEPLOYMENT = 'node-jenkins-demo'
    }

    stages {
        stage('Checkout code') {
            steps {
                git branch: 'main', url: 'https://github.com/manhcodee/node-jenkins-demo.git'
            }
        }

        stage('Build Docker image') {
            steps {
                sh """
                docker build -t ${DOCKER_IMAGE}:latest .
                """
            }
        }

        stage('Login to Docker Hub') {
            steps {
                sh """
                echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin
                """
            }
        }

        stage('Push Docker image') {
            steps {
                sh """
                docker push ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                kubectl set image deployment/${K8S_DEPLOYMENT} ${K8S_DEPLOYMENT}=${DOCKER_IMAGE}:latest -n ${K8S_NAMESPACE}
                kubectl rollout status deployment/${K8S_DEPLOYMENT} -n ${K8S_NAMESPACE}
                """
            }
        }
    }

    post {
        failure {
            echo 'Build failed!'
        }
        success {
            echo 'Build and deploy succeeded!'
        }
    }
}

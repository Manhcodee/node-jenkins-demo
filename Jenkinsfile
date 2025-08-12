pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Manhcodee/node-jenkins-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t node-jenkins-demo:latest .'
                }
            }
        }

        stage('Run Container (Local)') {
            steps {
                script {
                    // Xóa container cũ nếu tồn tại
                    sh 'docker rm -f node-jenkins-demo || true'
                    // Chạy container mới
                    sh 'docker run -d -p 3000:3000 --name node-jenkins-demo node-jenkins-demo:latest'
                }
            }
        }
    }
}

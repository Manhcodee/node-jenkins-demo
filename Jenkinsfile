pipeline {
    agent any

    stages {
        stage('Clone code') {
            steps {
                git branch: 'main', url: 'https://github.com/Manhcodee/node-jenkins-demo.git'
            }
        }

        stage('Build & Push with Kaniko') {
            steps {
                script {
                    sh """
                    kubectl run kaniko \
                      --rm -i --restart=Never \
                      --image=gcr.io/kaniko-project/executor:latest \
                      --serviceaccount=default \
                      --namespace=jenkins \
                      -- \
                      --context=git://github.com/Manhcodee/node-jenkins-demo.git \
                      --dockerfile=Dockerfile \
                      --destination=ghcr.io/manhcodee/node-jenkins-demo:latest \
                      --cleanup \
                      --skip-tls-verify \
                      --verbosity=debug \
                      --docker-config=/kaniko/.docker/
                    """
                }
            }
        }
    }
}

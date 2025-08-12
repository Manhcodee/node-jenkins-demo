pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command:
    - cat
    tty: true
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker/
  volumes:
    - name: docker-config
      secret:
        secretName: docker-config-secret
"""
        }
    }
    stages {
        stage('Build & Push with Kaniko') {
            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                      --context=git://github.com/Manhcodee/node-jenkins-demo.git \
                      --dockerfile=Dockerfile \
                      --destination=ghcr.io/manhcodee/node-jenkins-demo:latest \
                      --skip-tls-verify
                    """
                }
            }
        }
    }
}

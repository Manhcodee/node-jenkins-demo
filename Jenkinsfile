pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: kaniko
spec:
  containers:
  - name: kaniko
    image: docker.io/kaniko-project/executor:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker/
  volumes:
  - name: kaniko-secret
    secret:
      secretName: docker-config-secret
"""
        }
    }
    stages {
        stage('Build & Push Docker Image') {
            steps {
                container('kaniko') {
                    sh '''
                    /kaniko/executor \
                      --dockerfile=Dockerfile \
                      --context=./ \
                      --destination=your-dockerhub-username/node-jenkins-demo:latest \
                      --cleanup
                    '''
                }
            }
        }
    }
}

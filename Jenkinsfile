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
  serviceAccountName: jenkins
  containers:
    - name: kaniko
      image: gcr.io/kaniko-project/executor:latest
      tty: true
      volumeMounts:
        - name: docker-config
          mountPath: /kaniko/.docker/
    - name: kubectl
      image: bitnami/kubectl:latest
      command: ["sh", "-c", "sleep infinity"]
      tty: true
  volumes:
    - name: docker-config
      secret:
        secretName: docker-config-secret
"""
    }
  }

  environment {
    REGISTRY   = "ghcr.io"
    IMAGE_REPO = "manhcodee/node-jenkins-demo"
    IMAGE_TAG  = "latest"
    K8S_NS     = "node-demo"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build & Push (Kaniko)') {
      steps {
        container('kaniko') {
          sh '''
            /kaniko/executor \
              --context="$WORKSPACE" \
              --dockerfile="$WORKSPACE/Dockerfile" \
              --destination=${REGISTRY}/${IMAGE_REPO}:${IMAGE_TAG} \
              --destination=${REGISTRY}/${IMAGE_REPO}:${BUILD_NUMBER} \
              --skip-tls-verify \
              --cache=true
          '''
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        container('kubectl') {
          sh '''
            kubectl apply -f k8s/namespace.yaml
            kubectl apply -f k8s/deployment.yaml -n ${K8S_NS}
            kubectl apply -f k8s/service.yaml    -n ${K8S_NS}
            kubectl rollout status deployment/node-jenkins-demo -n ${K8S_NS}
            kubectl get svc node-jenkins-demo-service -n ${K8S_NS}
          '''
        }
      }
    }
  }
}

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
  tolerations:
  - key: "node-role.kubernetes.io/control-plane"
    operator: "Exists"
    effect: "NoSchedule"
  containers:
    - name: jnlp
      image: jenkins/inbound-agent:latest
      args: ['\$(JENKINS_SECRET)', '\$(JENKINS_NAME)']
      resources:
        requests:
          memory: "256Mi"
          cpu: "0.2"
        limits:
          memory: "512Mi"
          cpu: "0.5"
      imagePullPolicy: IfNotPresent
    - name: kaniko
      image: gcr.io/kaniko-project/executor:latest
      tty: true
      volumeMounts:
        - name: docker-config
          mountPath: /kaniko/.docker/
      resources:
        requests:
          memory: "512Mi"
          cpu: "0.5"
        limits:
          memory: "1Gi"
          cpu: "1"
      imagePullPolicy: IfNotPresent
    - name: kubectl
      image: bitnami/kubectl:latest
      command: ["sh", "-c", "sleep infinity"]
      tty: true
      resources:
        requests:
          memory: "128Mi"
          cpu: "0.1"
        limits:
          memory: "256Mi"
          cpu: "0.2"
      imagePullPolicy: IfNotPresent
  volumes:
    - name: docker-config
      secret:
        secretName: docker-config-secret
  imagePullSecrets:
    - name: docker-config-secret
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
      steps {
        checkout scm
      }
    }

    stage('Build & Push (Kaniko)') {
      steps {
        container('kaniko') {
          sh '''
            echo "Registry: ${REGISTRY}"
            echo "Image Repo: ${IMAGE_REPO}"
            echo "Image Tag: ${IMAGE_TAG}"
            echo "Build Number: ${env.BUILD_NUMBER}"
            /kaniko/executor \
              --context="$WORKSPACE" \
              --dockerfile="$WORKSPACE/Dockerfile" \
              --destination="${REGISTRY}/${IMAGE_REPO}:${IMAGE_TAG}" \
              --destination="${REGISTRY}/${IMAGE_REPO}:${env.BUILD_NUMBER}" \
              --cache=true \
              --verbosity=debug
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

  post {
    always {
      echo 'Pipeline completed.'
    }
    success {
      echo 'Pipeline succeeded!'
    }
    failure {
      echo 'Pipeline failed. Check logs for details.'
    }
  }
}
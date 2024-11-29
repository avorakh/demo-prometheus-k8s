pipeline {
    agent {
        kubernetes {
            label 'helm-kubectl'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: helm-kubectl
    image: alpine/helm:3.7.1
    command:
    - cat
    tty: true
"""
        }
    }

    environment {
        KUBECONFIG_CREDENTIALS_ID = 'k8s-credentials'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Kubernetes Context') {
            steps {
                script {
                    withCredentials([file(credentialsId: env.KUBECONFIG_CREDENTIALS_ID, variable: 'KUBECONFIG_FILE')]) {
                        sh 'mkdir -p /root/.kube'
                        sh 'cp $KUBECONFIG_FILE /root/.kube/config'
                    }
                }
            }
        }

        stage('Install Prometheus') {
            steps {
                container('helm-kubectl') {
                    script {
                        sh 'kubectl create namespace demo-metrics || true'
                        sh 'helm repo add bitnami https://charts.bitnami.com/bitnami'
                        sh 'helm repo update'
                        sh 'helm upgrade --install demo-prometheus bitnami/kube-prometheus -n demo-metrics'
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'rm -f /root/.kube/config'
        }
    }
}
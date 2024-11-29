pipeline {
    agent {
        kubernetes {
            label 'helm-agent'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: helm
    image: alpine/helm:3.7.1
    command:
    - cat
    tty: true
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true
"""
        }
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Install Prometheus') {
            steps {
                container('helm') {
                    script {
                        sh 'helm repo add bitnami https://charts.bitnami.com/bitnami'
                        sh 'helm repo update'
                        sh 'helm upgrade --install demo-prometheus bitnami/kube-prometheus -n demo-metrics --create-namespace'
                    }
                }
            }
        }
    }
}
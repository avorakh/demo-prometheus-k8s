pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: helm
    image: alpine/helm:3.11.1
    command: ['cat']
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
                        sh 'helm repo add bitnami https://charts.bitnami.com/bitnami'
                        sh 'helm repo update'
                        sh ' helm list -n jenkins'
                        sh 'helm upgrade --install demo-prometheus bitnami/kube-prometheus -n demo-metrics'
                }
            }
            }
        }
    }
}
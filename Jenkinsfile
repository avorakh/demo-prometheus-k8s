pipeline {
    agent any

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
                        sh 'mkdir -p $HOME/.kube'
                        sh 'cp $KUBECONFIG_FILE $HOME/.kube/config'
                    }
                }
            }
        }

        stage('Install Prometheus') {
            steps {
                script {
                    sh 'kubectl create namespace demo-metrics || true'
                    
                    sh 'helm repo add bitnami https://charts.bitnami.com/bitnami'
                    sh 'helm repo update'
                    sh 'helm upgrade --install demo-prometheus bitnami/kube-prometheus -n demo-metrics'
                }
            }
        }
    }

    post {
        always {
            sh 'rm -f $HOME/.kube/config'
        }
    }
}
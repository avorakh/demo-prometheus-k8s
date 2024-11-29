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

        stage('Install Tools') {
            steps {
                script {
                    sh '''
                    if ! command -v kubectl &> /dev/null; then
                        curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
                        chmod +x ./kubectl
                        mv ./kubectl /usr/local/bin/kubectl
                    fi
                    '''

                    sh '''
                    if ! command -v helm &> /dev/null; then
                        curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
                    fi
                    '''
                }
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
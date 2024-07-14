pipeline {
    agent any

    environment {
        REGISTRY = 'index.docker.io'
        REGISTRY_CREDENTIALS = 'docker-registry-credentials' // Jenkins credential ID for Docker registry
        KUBE_CONFIG = credentials('kube-config') // Jenkins credential ID for kubeconfig
        IMAGE_NAME = 'chrisgarcia0/avaliacao-infra-app'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("https://${env.REGISTRY}", env.REGISTRY_CREDENTIALS) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    writeFile file: 'kubeconfig', text: env.KUBE_CONFIG
                    withKubeConfig([credentialsId: 'kube-config', contextName: 'my-cluster-context', serverUrl: 'https://k8s-api-server:6443']) {
                        sh 'kubectl apply -f deployment.yaml'
                        sh 'kubectl apply -f service.yaml'
                        sh 'kubectl apply -f ingress.yaml'
                        sh 'kubectl apply -f persistent-volume.yaml'
                        sh 'kubectl apply -f secret.yaml'
                        sh 'kubectl apply -f configmap.yaml'
                        sh 'kubectl apply -f mongo-deployment.yaml'
                        sh 'kubectl apply -f mongo-service.yaml'
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

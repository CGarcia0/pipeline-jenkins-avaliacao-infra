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
                    echo "Iniciando Checkout do código-fonte"
                    checkout scm
                    echo "Checkout concluído com sucesso"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Iniciando build da imagem Docker"
                    def dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    echo "Imagem Docker construída com sucesso: ${dockerImage.imageName()}"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "Iniciando login no Docker Registry"
                    docker.withRegistry("https://${env.REGISTRY}", env.REGISTRY_CREDENTIALS) {
                        echo "Login bem-sucedido, iniciando push da imagem Docker"
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Iniciando deploy no Kubernetes"
                    writeFile file: 'kubeconfig', text: env.KUBE_CONFIG
                    withKubeConfig([credentialsId: 'kube-config', contextName: 'my-cluster-context', serverUrl: 'https://k8s-api-server:6443']) {
                        sh 'kubectl apply -f app-deployment.yaml'
                        sh 'kubectl apply -f app-service.yaml'
                        sh 'kubectl apply -f current-configmap.yaml'
                        sh 'kubectl apply -f docker-secret.yaml'
                        sh 'kubectl apply -f ingress.yaml'
                        sh 'kubectl apply -f mongo-deployment.yaml'
                        sh 'kubectl apply -f mongo-service.yaml'
                        sh 'kubectl apply -f persistent-volume.yaml'
                        sh 'kubectl apply -f pvc.yaml'
                        sh 'kubectl apply -f secret.yaml'
                    }
                    echo "Deploy no Kubernetes concluído com sucesso"
                }
            }
        }
    }

    post {
        always {
            echo "Limpando workspace"
            cleanWs()
        }
        failure {
            script {
                echo "A pipeline falhou. Verifique os logs acima para mais detalhes."
            }
        }
    }
}

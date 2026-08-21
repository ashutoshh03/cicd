pipeline {
    agent any

    environment {
        IMAGE_NAME = 'hello-world:latest'
        DEPLOYMENT_FILE = 'k8s/deployment.yaml'
        SERVICE_FILE = 'k8s/service.yaml'
    }

    stages {
        stage('Prepare Build Environment') {
            steps {
                sh '''
                    set -eux
                    apt-get update
                    apt-get install -y curl ca-certificates gnupg lsb-release

                    curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
                    apt-get install -y nodejs docker.io

                    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

                    node -v
                    npm -v
                    docker version
                    kubectl version --client
                '''
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ashutoshh03/cicd.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    set -eux
                    eval $(minikube docker-env)
                    docker build -t ${IMAGE_NAME} .
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    set -eux
                    kubectl apply -f ${DEPLOYMENT_FILE}
                    kubectl apply -f ${SERVICE_FILE}
                    kubectl rollout status deployment/hello-world --timeout=180s
                    kubectl get pods -o wide
                    kubectl get svc hello-world
                '''
            }
        }

        stage('Verify Self-Healing') {
            steps {
                sh '''
                    set -eux
                    POD_NAME=$(kubectl get pod -l app=hello-world -o jsonpath="{.items[0].metadata.name}")
                    echo "Deleting pod: ${POD_NAME}"
                    kubectl delete pod ${POD_NAME}
                    sleep 20
                    kubectl get pods -l app=hello-world -o wide
                    kubectl rollout status deployment/hello-world --timeout=180s
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    set -eux
                    MINIKUBE_IP=$(minikube ip)
                    echo "Application URL: http://${MINIKUBE_IP}:30000"
                    curl -sSf "http://${MINIKUBE_IP}:30000/"
                    echo
                    curl -sSf "http://${MINIKUBE_IP}:30000/health"
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Deployment successful and Kubernetes self-healing verified.'
        }
        failure {
            echo 'Pipeline failed. Check build logs and Kubernetes status.'
        }
    }
}

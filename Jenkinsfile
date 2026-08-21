pipeline {
    agent any

    environment {
        IMAGE_NAME = 'hello-world:latest'
        DEPLOYMENT_FILE = 'k8s/deployment.yaml'
        SERVICE_FILE = 'k8s/service.yaml'
    }

    stages {
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
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    set -e
                    eval $(minikube docker-env)
                    docker build -t ${IMAGE_NAME} .
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
                    set -e
                    POD_NAME=$(kubectl get pod -l app=hello-world -o jsonpath="{.items[0].metadata.name}")
                    echo "Deleting pod: ${POD_NAME}"
                    kubectl delete pod ${POD_NAME}
                    sleep 15
                    kubectl get pods -l app=hello-world -o wide
                    kubectl rollout status deployment/hello-world --timeout=180s
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    set -e
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

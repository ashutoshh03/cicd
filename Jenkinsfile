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
                    
                    # Install kubectl via apt package manager
                    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | apt-key add -
                    echo "deb https://pkgs.k8s.io/core:/stable:/v1.31/deb /" | tee /etc/apt/sources.list.d/kubernetes.list
                    apt-get update
                    apt-get install -y kubectl
                    
                    # Verify
                    node -v
                    npm -v
                    docker --version
                    kubectl version --client
                '''
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
            docker build -t ${IMAGE_NAME} .
        '''
    }
}

stage('Deploy to Kubernetes') {
    steps {
        sh '''
            set -eux
            export KUBECONFIG=/root/.kube/config
            kubectl apply -f ${DEPLOYMENT_FILE}
            kubectl apply -f ${SERVICE_FILE}
            kubectl rollout status deployment/hello-world --timeout=180s
        '''
    }
}

        stage('Verify Self-Healing') {
            steps {
                sh '''
                    set -eux
                    POD_NAME=$(kubectl get pod -l app=hello-world -o jsonpath="{.items[0].metadata.name}")
                    kubectl delete pod ${POD_NAME}
                    sleep 20
                    kubectl rollout status deployment/hello-world --timeout=180s
                '''
            }
        }
    }
}
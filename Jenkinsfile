pipeline {
    agent any

    environment {
        // macOS paths
        PATH = "/usr/local/bin:/opt/homebrew/bin:/usr/bin:/bin:/usr/sbin:/sbin"

        APP_NAME = 'hello-world'
        IMAGE_NAME = 'hello-world'

        DEPLOYMENT_FILE = 'k8s/deployment.yaml'
        SERVICE_FILE = 'k8s/service.yaml'

        K8S_CONTEXT = 'minikube'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub...'

                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                sh '''
                    set -eux

                    echo "========================================"
                    echo "VERIFYING ENVIRONMENT"
                    echo "========================================"

                    echo ""
                    echo "Node.js:"
                    which node
                    node --version

                    echo ""
                    echo "NPM:"
                    which npm
                    npm --version

                    echo ""
                    echo "Docker:"
                    which docker
                    docker --version

                    echo ""
                    echo "kubectl:"
                    which kubectl
                    kubectl version --client

                    echo ""
                    echo "Minikube:"
                    which minikube
                    minikube version

                    echo ""
                    echo "Kubernetes context:"
                    kubectl config current-context

                    echo ""
                    echo "Minikube status:"
                    minikube status

                    echo ""
                    echo "Kubernetes nodes:"
                    kubectl get nodes
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    set -eux

                    echo "Installing dependencies..."

                    npm ci
                '''
            }
        }

        stage('Unit Test') {
            steps {
                sh '''
                    set -eux

                    echo "Running unit tests..."

                    npm test
                '''
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                    set -eux

                    echo "Building application..."

                    npm run build
                '''
            }
        }

        stage('Archive Artifact') {
            steps {
                sh '''
                    set -eux

                    echo "Creating application artifact..."

                    rm -rf artifact
                    mkdir -p artifact

                    cp package.json artifact/
                    cp package-lock.json artifact/

                    if [ -d "src" ]; then
                        cp -R src artifact/
                    fi

                    echo "Artifact contents:"
                    find artifact -type f
                '''

                archiveArtifacts(
                    artifacts: 'artifact/**',
                    fingerprint: true
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -eux

                    echo "========================================"
                    echo "BUILDING DOCKER IMAGE"
                    echo "========================================"

                    echo "Building image inside Minikube..."

                    minikube image build \
                        -t ${IMAGE_NAME}:${BUILD_NUMBER} .

                    echo ""
                    echo "Image built:"
                    echo "${IMAGE_NAME}:${BUILD_NUMBER}"

                    echo ""
                    echo "Checking image in Minikube:"

                    minikube image ls | grep "${IMAGE_NAME}" || true
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    set -eux

                    echo "========================================"
                    echo "DEPLOYING TO KUBERNETES"
                    echo "========================================"

                    kubectl config use-context ${K8S_CONTEXT}

                    echo ""
                    echo "Applying Deployment..."

                    kubectl apply \
                        -f ${DEPLOYMENT_FILE}

                    echo ""
                    echo "Applying Service..."

                    kubectl apply \
                        -f ${SERVICE_FILE}

                    echo ""
                    echo "Updating image..."

                    kubectl set image \
                        deployment/${APP_NAME} \
                        ${APP_NAME}=${IMAGE_NAME}:${BUILD_NUMBER}

                    echo ""
                    echo "Waiting for rollout..."

                    kubectl rollout status \
                        deployment/${APP_NAME} \
                        --timeout=180s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -eux

                    echo "========================================"
                    echo "VERIFYING DEPLOYMENT"
                    echo "========================================"

                    echo ""
                    echo "Deployment:"
                    kubectl get deployment ${APP_NAME}

                    echo ""
                    echo "Pods:"
                    kubectl get pods \
                        -l app=${APP_NAME} \
                        -o wide

                    echo ""
                    echo "Service:"
                    kubectl get service ${APP_NAME}

                    echo ""
                    echo "Replica count:"
                    kubectl get deployment ${APP_NAME} \
                        -o jsonpath='{.status.replicas}'

                    echo ""

                    echo "Ready replicas:"
                    kubectl get deployment ${APP_NAME} \
                        -o jsonpath='{.status.readyReplicas}'

                    echo ""
                '''
            }
        }

        stage('Verify Self-Healing') {
            steps {
                sh '''
                    set -eux

                    echo "========================================"
                    echo "VERIFYING SELF-HEALING"
                    echo "========================================"

                    echo ""
                    echo "Pods before deletion:"

                    kubectl get pods \
                        -l app=${APP_NAME}

                    echo ""
                    echo "Selecting pod..."

                    POD_NAME=$(kubectl get pods \
                        -l app=${APP_NAME} \
                        -o jsonpath="{.items[0].metadata.name}")

                    echo "Deleting pod: ${POD_NAME}"

                    kubectl delete pod ${POD_NAME}

                    echo ""
                    echo "Waiting for Kubernetes to recreate the pod..."

                    sleep 10

                    echo ""
                    echo "Pods after deletion:"

                    kubectl get pods \
                        -l app=${APP_NAME} \
                        -o wide

                    echo ""
                    echo "Waiting for deployment..."

                    kubectl rollout status \
                        deployment/${APP_NAME} \
                        --timeout=180s

                    echo ""
                    echo "Final pod status:"

                    kubectl get pods \
                        -l app=${APP_NAME} \
                        -o wide
                '''
            }
        }
    }

    post {

        success {
            echo '''
========================================
CI/CD PIPELINE SUCCESSFUL
========================================

GitHub Checkout       : SUCCESS
Dependencies          : SUCCESS
Unit Tests            : SUCCESS
Application Build     : SUCCESS
Artifact              : CREATED
Docker Image          : CREATED
Kubernetes Deployment : SUCCESS
Replicas              : VERIFIED
Self-Healing          : VERIFIED
'''
        }

        failure {
            echo '''
========================================
CI/CD PIPELINE FAILED
========================================

Check the failed stage in the Jenkins console.
'''
        }

        always {
            echo 'Pipeline finished.'
        }
    }
}
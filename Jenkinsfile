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
                    echo "Starting Minikube if not already running..."
                    
                    if ! minikube status | grep -q "host: Running"; then
                        echo "Minikube is not running, starting it..."
                        minikube start --driver=docker || minikube start
                    else
                        echo "Minikube is already running"
                    fi

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
                    echo "VERIFYING SELF-HEALING (Replica Management)"
                    echo "========================================"

                    echo ""
                    echo "Initial pod count:"
                    INITIAL_PODS=$(kubectl get pods -l app=${APP_NAME} --no-headers | wc -l)
                    echo "Total pods: $INITIAL_PODS"

                    kubectl get pods -l app=${APP_NAME} -o wide

                    if [ "$INITIAL_PODS" -lt 2 ]; then
                        echo "WARNING: Expected at least 2 replicas, but found $INITIAL_PODS"
                    fi

                    echo ""
                    echo "Selecting a pod to delete..."

                    POD_NAME=$(kubectl get pods \
                        -l app=${APP_NAME} \
                        -o jsonpath="{.items[0].metadata.name}")

                    echo "Pod selected for deletion: ${POD_NAME}"

                    echo ""
                    echo "Pod details before deletion:"
                    kubectl describe pod ${POD_NAME} | grep -A 5 "Status:"

                    echo ""
                    echo "Deleting pod: ${POD_NAME}"

                    kubectl delete pod ${POD_NAME} --grace-period=5

                    echo ""
                    echo "Waiting for Kubernetes to recreate pods (self-healing in action)..."

                    sleep 15

                    echo ""
                    echo "Pod status after deletion (should show new pod):"

                    kubectl get pods -l app=${APP_NAME} -o wide

                    echo ""
                    echo "Checking pod creation timestamp:"
                    kubectl get pods -l app=${APP_NAME} -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.creationTimestamp}{"\n"}{end}'

                    echo ""
                    echo "Waiting for all replicas to be ready..."

                    kubectl rollout status deployment/${APP_NAME} --timeout=180s

                    echo ""
                    echo "Final deployment status:"
                    kubectl get deployment ${APP_NAME} -o jsonpath='{.status.replicas} replicas, {.status.readyReplicas} ready'
                    echo ""

                    echo "✓ Self-healing verification completed successfully!"
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
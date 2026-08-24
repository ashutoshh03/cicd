pipeline {
    agent any

    environment {
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

        // stage('Verify Environment') {
        //     steps {
        //         powershell '''
        //             $ErrorActionPreference = "Stop"

        //             Write-Host "========================================"
        //             Write-Host "VERIFYING ENVIRONMENT"
        //             Write-Host "========================================"

        //             Write-Host ""
        //             Write-Host "Node.js:"
        //             Get-Command node
        //             node --version

        //             Write-Host ""
        //             Write-Host "NPM:"
        //             Get-Command npm
        //             npm --version

        //             Write-Host ""
        //             Write-Host "Docker:"
        //             Get-Command docker
        //             docker --version

        //             Write-Host ""
        //             Write-Host "kubectl:"
        //             Get-Command kubectl
        //             kubectl version --client

        //             Write-Host ""
        //             Write-Host "Minikube:"
        //             Get-Command minikube
        //             minikube version

        //             Write-Host ""
        //             Write-Host "Starting Minikube if not already running..."

        //             $status = minikube status --output=json 2>$null

        //             if ($LASTEXITCODE -ne 0) {
        //                 Write-Host "Minikube is not running, starting it..."
        //                 minikube start --driver=docker

        //                 if ($LASTEXITCODE -ne 0) {
        //                     minikube start
        //                 }
        //             }
        //             else {
        //                 $statusJson = $status | ConvertFrom-Json

        //                 if ($statusJson.Host -and $statusJson.Host.Status -eq "Running") {
        //                     Write-Host "Minikube is already running"
        //                 }
        //                 else {
        //                     Write-Host "Minikube is not running, starting it..."
        //                     minikube start --driver=docker
        //                 }
        //             }

        //             Write-Host ""
        //             Write-Host "Kubernetes context:"
        //             kubectl config current-context

        //             Write-Host ""
        //             Write-Host "Minikube status:"
        //             minikube status

        //             Write-Host ""
        //             Write-Host "Kubernetes nodes:"
        //             kubectl get nodes
        //         '''
        //     }
        // }

        stage('Install Dependencies') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    Write-Host "Installing dependencies..."
                    npm ci
                '''
            }
        }

        stage('Unit Test') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    Write-Host "Running unit tests..."
                    npm test
                '''
            }
        }

        stage('Build Application') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    Write-Host "Building application..."
                    npm run build
                '''
            }
        }

        stage('Archive Artifact') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    Write-Host "Creating application artifact..."

                    if (Test-Path "artifact") {
                        Remove-Item "artifact" -Recurse -Force
                    }

                    New-Item -ItemType Directory -Path "artifact" | Out-Null

                    Copy-Item "package.json" "artifact/"
                    Copy-Item "package-lock.json" "artifact/"

                    if (Test-Path "src") {
                        Copy-Item "src" "artifact/" -Recurse
                    }

                    Write-Host "Artifact contents:"
                    Get-ChildItem "artifact" -Recurse
                '''

                archiveArtifacts(
                    artifacts: 'artifact/**',
                    fingerprint: true
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    Write-Host "========================================"
                    Write-Host "BUILDING DOCKER IMAGE"
                    Write-Host "========================================"

                    Write-Host "Building image inside Minikube..."

                    $tag = "$env:IMAGE_NAME`:$env:BUILD_NUMBER"

                    minikube image build -t $tag .

                    Write-Host ""
                    Write-Host "Image built:"
                    Write-Host $tag

                    Write-Host ""
                    Write-Host "Checking image in Minikube:"

                    minikube image ls | Select-String $env:IMAGE_NAME
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    Write-Host "========================================"
                    Write-Host "DEPLOYING TO KUBERNETES"
                    Write-Host "========================================"

                    kubectl config use-context $env:K8S_CONTEXT

                    Write-Host ""
                    Write-Host "Applying Deployment..."

                    kubectl apply -f $env:DEPLOYMENT_FILE

                    Write-Host ""
                    Write-Host "Applying Service..."

                    kubectl apply -f $env:SERVICE_FILE

                    Write-Host ""
                    Write-Host "Updating image..."

                    $image = "$env:IMAGE_NAME`:$env:BUILD_NUMBER"

                    kubectl set image `
                        "deployment/$env:APP_NAME" `
                        "$env:APP_NAME=$image"

                    Write-Host ""
                    Write-Host "Waiting for rollout..."

                    kubectl rollout status `
                        "deployment/$env:APP_NAME" `
                        --timeout=180s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    Write-Host "========================================"
                    Write-Host "VERIFYING DEPLOYMENT"
                    Write-Host "========================================"

                    Write-Host ""
                    Write-Host "Deployment:"
                    kubectl get deployment $env:APP_NAME

                    Write-Host ""
                    Write-Host "Pods:"
                    kubectl get pods `
                        -l "app=$env:APP_NAME" `
                        -o wide

                    Write-Host ""
                    Write-Host "Service:"
                    kubectl get service $env:APP_NAME

                    Write-Host ""
                    Write-Host "Replica count:"
                    kubectl get deployment $env:APP_NAME `
                        -o jsonpath='{.status.replicas}'

                    Write-Host ""

                    Write-Host "Ready replicas:"
                    kubectl get deployment $env:APP_NAME `
                        -o jsonpath='{.status.readyReplicas}'

                    Write-Host ""
                '''
            }
        }

        stage('Verify Self-Healing') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    Write-Host "========================================"
                    Write-Host "VERIFYING SELF-HEALING"
                    Write-Host "========================================"

                    Write-Host ""
                    Write-Host "Initial pod count:"

                    $pods = @(kubectl get pods `
                        -l "app=$env:APP_NAME" `
                        --no-headers)

                    $initialPods = $pods.Count

                    Write-Host "Total pods: $initialPods"

                    kubectl get pods `
                        -l "app=$env:APP_NAME" `
                        -o wide

                    if ($initialPods -lt 2) {
                        Write-Host "WARNING: Expected at least 2 replicas, but found $initialPods"
                    }

                    Write-Host ""
                    Write-Host "Selecting a pod to delete..."

                    $podName = kubectl get pods `
                        -l "app=$env:APP_NAME" `
                        -o jsonpath='{.items[0].metadata.name}'

                    if ([string]::IsNullOrWhiteSpace($podName)) {
                        throw "No pod found for application $env:APP_NAME"
                    }

                    Write-Host "Pod selected for deletion: $podName"

                    Write-Host ""
                    Write-Host "Pod details before deletion:"

                    kubectl describe pod $podName

                    Write-Host ""
                    Write-Host "Deleting pod: $podName"

                    kubectl delete pod $podName --grace-period=5

                    Write-Host ""
                    Write-Host "Waiting for Kubernetes to recreate pod..."

                    Start-Sleep -Seconds 15

                    Write-Host ""
                    Write-Host "Pod status after deletion:"

                    kubectl get pods `
                        -l "app=$env:APP_NAME" `
                        -o wide

                    Write-Host ""
                    Write-Host "Checking pod creation timestamps:"

                    kubectl get pods `
                        -l "app=$env:APP_NAME" `
                        -o custom-columns="NAME:.metadata.name,CREATED:.metadata.creationTimestamp"

                    Write-Host ""
                    Write-Host "Waiting for all replicas to be ready..."

                    kubectl rollout status `
                        "deployment/$env:APP_NAME" `
                        --timeout=180s

                    Write-Host ""
                    Write-Host "Final deployment status:"

                    kubectl get deployment $env:APP_NAME

                    Write-Host ""
                    Write-Host "Self-healing verification completed successfully!"
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

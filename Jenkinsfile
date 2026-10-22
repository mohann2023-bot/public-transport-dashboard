pipeline {
    agent any
    
    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'v1.1', description: 'Docker Image Tag Version')
    }
    
    environment {
        DOCKER_HUB_USER = 'mohanmonish2006'
        IMAGE_NAME      = 'transport-dashboard'
        REGISTRY_CRED   = 'docker-hub-credentials'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Parallel Build & Security Scan') {
            parallel {
                stage('Build Docker Image') {
                    steps {
                        script {
                            dockerImage = docker.build("/:")
                        }
                    }
                }
                stage('Static Mock Code Validation') {
                    steps {
                        echo "Validating code syntax standards..."
                        // Using native PowerShell to match string contents instead of Linux grep
                        powershell "Select-String -Pattern '<html>' -Path index.html"
                    }
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://docker.io', "") {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                    // Using native PowerShell commands to update the YAML image text dynamically
                    powershell "(Get-Content deployment.yaml) -replace 'image: .*', 'image: /:' | Set-Content deployment.yaml"
                    powershell "kubectl --kubeconfig='' apply -f deployment.yaml"
                    powershell "kubectl --kubeconfig='' rollout status deployment/transport-dashboard --timeout=60s"
                }
            }
        }
    }
    
    post {
        failure {
            echo "? Deployment update failed! Triggering automatic rollback procedure..."
            withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                powershell "kubectl --kubeconfig='' rollout undo deployment/transport-dashboard"
                powershell "kubectl --kubeconfig='' rollout status deployment/transport-dashboard"
            }
        }
        success {
            echo "? Application successfully deployed/updated."
        }
    }
}

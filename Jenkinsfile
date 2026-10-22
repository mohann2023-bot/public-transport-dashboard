pipeline {
    agent any
    
    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'v1.1', description: 'Docker Image Tag Version')
    }
    
    environment {
        DOCKER_HUB_USER = 'mohanmonish2006'
        IMAGE_NAME      = 'transportdashboard'
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
                        // Native Windows terminal build execution step
                        bat "docker build -t %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG% ."
                    }
                }
                stage('Static Mock Code Validation') {
                    steps {
                        echo "Validating code syntax standards..."
                        powershell "Select-String -Pattern '<html>' -Path index.html"
                    }
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                // Dynamically binds your stored web credentials parameters safely to local variables
                withCredentials([usernamePassword(credentialsId: "", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USER')]) {
                    // Logs in and pushes cleanly using raw system commands
                    bat "docker login -u %DOCKER_USER% -p %DOCKER_PASSWORD%"
                    bat "docker push %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG%"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
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

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
                            dockerImage = docker.build("\/\:\")
                        }
                    }
                }
                stage('Static Mock Code Validation') {
                    steps {
                        echo "Validating code syntax standards..."
                        sh "grep -q '<html>' index.html"
                    }
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://docker.io', "\") {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                    // Update yaml manifest dynamically with target version tag
                    sh "sed -i 's|image: .*|image: \/\:\|g' deployment.yaml"
                    
                    // Apply application rollout
                    sh "kubectl --kubeconfig=\ apply -f deployment.yaml"
                    
                    // Track rollout completion and timeout quickly if a failure occurs
                    sh "kubectl --kubeconfig=\ rollout status deployment/transport-dashboard --timeout=60s"
                }
            }
        }
    }
    
    post {
        failure {
            echo "? Deployment update failed! Triggering automatic rollback procedure..."
            withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                sh "kubectl --kubeconfig=\ rollout undo deployment/transport-dashboard"
                sh "kubectl --kubeconfig=\ rollout status deployment/transport-dashboard"
            }
        }
        success {
            echo "? Application successfully deployed/updated."
        }
    }
}

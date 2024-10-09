pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID_CREDS = credentials('acesskey') // Access Key ID credentials ID
        AWS_SECRET_ACCESS_KEY_CREDS = credentials('secretkey') // Secret Access Key credentials ID
        KUBE_CONFIG_CREDS = credentials('kubeconfig') 
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout your source code from GitHub
                script {
                    // Attempt to checkout the source code
                    git url: "https://github.com/Dharmendrasingh9760/TWSThreeTierAppChallenge.git", branch: "main"
                }
            }
        }

        stage('Build and Push Frontend Docker Image') {
            steps {
                script {
                    // Set AWS CLI environment variables
                    withEnv([
                        "AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID_CREDS}",
                        "AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY_CREDS}"
                    ]) {
                        // Configure AWS CLI
                        sh "aws configure set aws_access_key_id ${AWS_ACCESS_KEY_ID_CREDS}"
                        sh "aws configure set aws_secret_access_key ${AWS_SECRET_ACCESS_KEY_CREDS}"
                        sh "aws configure list"
                        sh "aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/e0f5y9k7"
                        sh "docker build -t three-repo  frontend"
                        sh "docker tag three-repo:latest public.ecr.aws/e0f5y9k7/three-repo:latest"
                        sh "docker push public.ecr.aws/e0f5y9k7/three-repo:latest"
                    }
                }
            }
        }

        stage('Build and Push Backend Docker Image') {
            steps {
                script {
                    // Build and push the backend Docker image to ECR
                    sh "docker build -t three-repobackend  backend"
                    sh "docker tag three-repobackend:latest public.ecr.aws/e0f5y9k7/three-repobackend:latest"
                    sh "docker push public.ecr.aws/e0f5y9k7/three-repobackend:latest"
                }
            }
        }
        
        stage('Apply MongoDB Configurations') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBE_CONFIG')]) {
                        sh "kubectl --kubeconfig=${KUBE_CONFIG} apply -f k8s_manifests/mongo/deploy.yaml -f k8s_manifests/mongo/secrets.yaml -f k8s_manifests/mongo/service.yaml"
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBE_CONFIG')]) {
                    sh "kubectl --kubeconfig=${KUBE_CONFIG} apply -f k8s_manifests/backend-deployment.yaml -f k8s_manifests/backend-service.yaml"
                    sh "kubectl --kubeconfig=${KUBE_CONFIG} apply -f k8s_manifests/frontend-deployment.yaml -f k8s_manifests/frontend-service.yaml"
                    sh "kubectl --kubeconfig=${KUBE_CONFIG} apply -f k8s_manifests/full_stack_lb.yaml"
                    sh "kubectl --kubeconfig=${KUBE_CONFIG} delete deployment api -n workshop"
                    sh "kubectl --kubeconfig=${KUBE_CONFIG} create -f k8s_manifests/backend-deployment.yaml"
                    sh "kubectl --kubeconfig=${KUBE_CONFIG} delete deployment frontend -n workshop"
                    sh "kubectl --kubeconfig=${KUBE_CONFIG} create -f k8s_manifests/frontend-deployment.yaml"
                }
                }
            }
        }
    }
}

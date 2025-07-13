pipeline {
    agent any
    
    tools {
    jdk 'jdk11'
    maven  'maven3.9.8'
  }  

    environment {
        // AWS & EKS Config
        AWS_ACCOUNT_ID = '951247596879'  // Replace with your AWS Account ID
        AWS_REGION = 'us-east-2'
        ECR_REPO = 'staging'
        DOCKER_IMAGE = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${BUILD_NUMBER}"
        
        // EKS Config
        EKS_CLUSTER_NAME = 'staging'
        KUBE_NAMESPACE = 'jenkins-project'
        
        // Git Config
        GIT_REPO = 'https://github.com/victorblinks/eks-jenkins-maven.git'
        BRANCH = 'demo'
    }

    stages {
        stage('Checkout Git') {
            steps {
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh "docker push ${DOCKER_IMAGE}"
            }
        }

        stage('Configure kubectl for EKS') {
            steps {
                sh "aws eks --region ${AWS_REGION} update-kubeconfig --name ${EKS_CLUSTER_NAME}"
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    // Apply Kubernetes manifests (assuming they are in `k8s/` dir)
                    sh "kubectl apply -f k8s/deploy_svc.yml -n ${KUBE_NAMESPACE}"
                    sh "kubectl apply -f k8s/service.yaml -n ${KUBE_NAMESPACE}"
                    
                    // (Optional) Blue-Green or Canary Deployment Logic
                    // sh "kubectl set image deployment/project-1-deployment app=${DOCKER_IMAGE} -n ${KUBE_NAMESPACE}"
                }
            }
        }
    }

    post {
        success {
            echo "Deployment to EKS succeeded! ✅"
        }
        failure {
            echo "Deployment failed! Check logs. ❌"
        }
    }
}
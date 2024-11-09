pipeline {
    agent any
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-credentials')  
        AWS_CREDENTIALS = credentials('aws-credentials')  
        EC2_UAT_IP = 'uat-ec2-ip'  
        EC2_PROD_IP = 'prod-ec2-ip'  
        IMAGE_NAME = 'dotnet-hello-world'
        DOCKER_TAG = "latest"
        AWS_SSH_USER = 'ec2-user'  
        DOCKER_REGISTRY = 'docker.io'  
        DOCKER_IMAGE = "${DOCKER_REGISTRY}/${IMAGE_NAME}:${DOCKER_TAG}"
    }
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['UAT', 'Production'], description: 'Choose the environment to deploy')
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/yourusername/your-repo-name.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }
        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    withDockerRegistry([ credentialsId: DOCKER_HUB_CREDENTIALS ]) {
                        docker.push("${DOCKER_IMAGE}")
                    }
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                script {
                    def ec2_ip = params.DEPLOY_ENV == 'UAT' ? EC2_UAT_IP : EC2_PROD_IP
                    sshagent([AWS_SSH_USER]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${AWS_SSH_USER}@${ec2_ip} "
                            docker pull ${DOCKER_IMAGE} &&
                            docker stop dotnet-hello-world || true &&
                            docker rm dotnet-hello-world || true &&
                            docker run -d --name dotnet-hello-world -p 80:80 ${DOCKER_IMAGE}
                            "
                        """
                    }
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                script {
                    def ec2_ip = params.DEPLOY_ENV == 'UAT' ? EC2_UAT_IP : EC2_PROD_IP
                    sh """
                        curl http://${ec2_ip}/health
                    """
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}


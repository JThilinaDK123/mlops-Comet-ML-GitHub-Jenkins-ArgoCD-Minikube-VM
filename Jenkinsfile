pipeline {
    agent any
    environment {
        DOCKER_HUB_REPO = "thilina9718/thilina-mlops"
        DOCKER_HUB_CREDENTIALS_ID = "dockerthilina"
        BUILD_NUMBER = "latest"
    }
    stages {
        stage('Checkout Github') {
            steps {
                echo 'Checking out code from GitHub...'
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-cred', url: 'https://github.com/JThilinaDK123/mlops-Comet-ML-GitHub-Jenkins-ArgoCD-Minikube-VM.git']])
            }
        }        

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    def dockerImage = docker.build("${DOCKER_HUB_REPO}:${env.BUILD_NUMBER}")
                    env.IMAGE_NAME = "${DOCKER_HUB_REPO}:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                script {
                    echo 'Pushing Docker image to DockerHub...'
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_HUB_CREDENTIALS_ID}") {
                        docker.image(env.IMAGE_NAME).push('latest')
                    }
                }
            }
        }

        // stage('Build Docker Image') {
        //     steps {
        //         script {
        //             echo 'Building Docker image...'
        //             dockerImage = docker.build("${DOCKER_HUB_REPO}:${env.BUILD_NUMBER}")
        //         }
        //     }
        // }
        // stage('Push Image to DockerHub') {
        //     steps {
        //         script {
        //             echo 'Pushing Docker image to DockerHub...'
        //             docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_HUB_CREDENTIALS_ID}") {
        //                 dockerImage.push('latest')
        //             }
        //         }
        //     }
        // }
        stage('Install Kubectl & ArgoCD CLI') {
            steps {
                echo 'Installing Kubectl and ArgoCD CLI...'
            }
        }
        stage('Apply Kubernetes & Sync App with ArgoCD') {
            steps {
                echo 'Applying Kubernetes and syncing with ArgoCD...'
            }
        }
    }
}

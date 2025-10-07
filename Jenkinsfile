pipeline {
    agent any
    environment {
        DOCKER_HUB_REPO = "thilina9718/thilina-mlops"
        DOCKER_HUB_CREDENTIALS_ID = "docker-cred"
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

        stage('Install Kubectl & ArgoCD CLI') {
            steps {
                sh '''
                echo 'installing Kubectl & ArgoCD cli...'
                curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                chmod +x kubectl
                mv kubectl /usr/local/bin/kubectl
                curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
                chmod +x /usr/local/bin/argocd
                '''
            }
        }
        
        stage('Apply Kubernetes & Sync App with Argo CD') {
            steps {
                script {
                    kubeconfig(credentialsId: 'kubeconfig-cred', serverUrl: 'https://192.168.49.2:8443'){
                        sh '''
                        argocd login 34.72.5.170:31056 --username admin --password $(kubectl get secret -n argocd 
                        argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d) --insecure
                        argocd app sync gitopsapp
                        '''
                    }
                }
            }
        }
    }
}

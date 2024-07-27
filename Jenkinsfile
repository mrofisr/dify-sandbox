pipeline {
    agent any
    stages {
        stage('Info') {
            steps {
                echo 'Starting the pipeline...'
            }
        }
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Get Version') {
            steps {
                script {
                    GIT_TAG = sh(script: "git describe --tags --match 'v[0-9]*' --abbrev=0 || echo 'latest'", returnStdout: true).trim()
                    echo "Version: ${GIT_TAG}"
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    sh "bash build/build_amd64.sh"
                    echo 'Building Dify Sandbox...'
                    sh "docker build -t ghcr.io/mrofisr/dify-sandbox:${GIT_TAG} docker/amd64"
                }
            }
        }
        stage('Docker Login') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'container_registry', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                        echo "Logging in to Docker registry"
                        sh "echo $DOCKER_PASSWORD | docker login ghcr.io -u $DOCKER_USER --password-stdin"
                    }
                }
            }
        }
        stage('Image Push') {
            steps {
                script {
                    echo 'Pushing Dify Sandbox image...'
                    sh "docker push ghcr.io/mrofisr/dify-sandbox:${GIT_TAG}"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'minikube', variable: 'minikube')]) {
                        sh "mkdir -p ~/.kube"
                        sh "cp $minikube ~/.kube/config"
                        sh "kubectl config use-context minikube"
                        sh "kubectl get nodes"
                        sh "rm -rf dify-kubernetes"
                        sh "git clone https://github.com/mrofisr/dify-kubernetes"
                        sh "sed -i 's|{{VERSION}}|${GIT_TAG}|g' dify-kubernetes/dify-sandbox/deployment.yaml"
                        sh "kubectl apply -f dify-kubernetes/dify-sandbox/deployment.yaml"
                    }
                }
            }
        }
    }
    post {
        success {
            echo "I will only say Hello if the pipeline is successful!"
        }
        failure {
            echo "I will only say Hello if the pipeline has failed!"
        }
    }
}

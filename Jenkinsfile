pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "mryum/todo-app"
        DOCKER_TAG   = "latest"
        REGISTRY_CREDS = credentials('dockerhub-creds')
    }
    stages {
        stage('Code Fetch') {
            steps {
                echo 'Fetching latest code from GitHub...'
                checkout scm
            }
        }
        stage('Docker Image Creation') {
            steps {
                echo 'Building Docker image...'
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    sh "echo ${REGISTRY_CREDS_PSW} | docker login -u ${REGISTRY_CREDS_USR} --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    echo 'Docker image pushed to DockerHub successfully!'
                }
            }
        }
        stage('Kubernetes Deployment') {
            steps {
                echo 'Deploying application to Kubernetes...'
                script {
                    sh "kubectl apply -f k8s/"
                    sh "kubectl rollout status deployment/todo-app"
                    echo 'Application deployed on Kubernetes!'
                }
            }
        }
        stage('Prometheus/Grafana Setup') {
            steps {
                echo 'Setting up monitoring stack...'
                script {
                    sh """
                        helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
                        helm repo update
                        helm upgrade --install monitoring prometheus-community/kube-prometheus-stack \
                            --namespace monitoring \
                            --create-namespace \
                            --set grafana.service.type=NodePort \
                            --set prometheus.service.type=NodePort
                    """
                    echo 'Monitoring stack deployed!'
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully! App is live.'
        }
        failure {
            echo 'Pipeline failed. Check the logs above for errors.'
        }
    }
}

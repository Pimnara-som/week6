pipeline {
    agent {
        label 'k8s-agent-1'
    }

    triggers {
        githubPush()
    }

    environment {
        APP_NAME    = 'my-nginx-web'
        IMAGE_TAG   = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${APP_NAME}:${IMAGE_TAG} -t ${APP_NAME}:latest web/"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh "kubectl apply -f nginx/deployment/nginx.yaml"
                    sh "kubectl apply -f nginx/service/nginx.yaml"
                    sh "kubectl apply -f nginx/ingress/ingress.yaml"
                    sh "kubectl set image deployment/nginx-deployment nginx-container=${APP_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    sh "kubectl rollout status deployment/nginx-deployment --timeout=120s"
                    sh "kubectl get pods -l app=my-nginx"
                    sh "kubectl get svc nginx-service"
                    sh "kubectl get ingress nginx-ingress"
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful! Access at http://my-nginx.local"
        }
        failure {
            echo "Deployment failed! Check logs for details."
        }
    }
}

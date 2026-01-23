pipeline {
    agent any

    environment {
        BACKEND_IMAGE   = "someone15me/dp:backend"
        FRONTEND_IMAGE  = "someone15me/dp:frontend"
        DEPLOYMENT_NAME = "task-management-app"
        K8S_NAMESPACE   = "task-management"
        BUILD_TAG       = "${env.BUILD_NUMBER}-${GIT_COMMIT.substring(0,7)}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                sh "docker build -t ${BACKEND_IMAGE}-${BUILD_TAG} ./Backend"
                sh "docker build -t ${FRONTEND_IMAGE}-${BUILD_TAG} ./Frontend"
            }
        }

        stage('Push Docker Images') {
            steps {
                sh "docker push ${BACKEND_IMAGE}-${BUILD_TAG}"
                sh "docker push ${FRONTEND_IMAGE}-${BUILD_TAG}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    # Ensure namespace exists
                    kubectl apply -f k8s/namespace.yaml

                    # Apply the deployment (create or update)
                    kubectl apply -f k8s/deployment.yaml -n ${K8S_NAMESPACE}

                    # Update images
                    kubectl set image deployment/${DEPLOYMENT_NAME} \\
                        backend=${BACKEND_IMAGE}-${BUILD_TAG} \\
                        frontend=${FRONTEND_IMAGE}-${BUILD_TAG} \\
                        -n ${K8S_NAMESPACE}

                    # Wait for rollout
                    kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${K8S_NAMESPACE}
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed with BUILD_TAG=${BUILD_TAG}"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}

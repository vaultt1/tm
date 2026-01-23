pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "someone15me/dp:backend"
        FRONTEND_IMAGE = "someone15me/dp:frontend"
        K8S_NAMESPACE = "task-management"
        DEPLOYMENT_NAME = "task-management-app"
        BUILD_TAG = "${env.BUILD_NUMBER}-${GIT_COMMIT.substring(0,7)}"
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
                kubectl set image deployment/${DEPLOYMENT_NAME} \
                  backend=${BACKEND_IMAGE}-${BUILD_TAG} \
                  frontend=${FRONTEND_IMAGE}-${BUILD_TAG} \
                  -n ${K8S_NAMESPACE}

                # Wait until rollout is complete
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

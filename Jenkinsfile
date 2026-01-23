pipeline {
    agent any

    environment {
        // Docker Hub info
        DOCKER_USER     = "someone15me"
        BUILD_TAG       = "${env.BUILD_NUMBER}"                // Unique tag per build
        BACKEND_IMAGE   = "${DOCKER_USER}/dp:backend-${BUILD_TAG}"
        FRONTEND_IMAGE  = "${DOCKER_USER}/dp:frontend-${BUILD_TAG}"

        // Kubernetes info
        K8S_NAMESPACE   = "task-management"
        K8S_DEPLOYMENT  = "task-management-app"
    }

    stages {

        stage('Checkout Source') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/vaultt1/tm.git'
            }
        }

        stage('Build Backend Image') {
            steps {
                sh """
                    docker build -t ${BACKEND_IMAGE} ./Backend
                """
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh """
                    docker build -t ${FRONTEND_IMAGE} ./Frontend
                """
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([string(credentialsId: 'DOCKERHUB_TOKEN', variable: 'DOCKER_TOKEN')]) {
                    sh "echo $DOCKER_TOKEN | docker login -u ${DOCKER_USER} --password-stdin"
                }
            }
        }

        stage('Push Images to DockerHub') {
            steps {
                sh """
                    docker push ${BACKEND_IMAGE}
                    docker push ${FRONTEND_IMAGE}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'KUBECONFIG_FILE', variable: 'KUBECONFIG')]) {
                    sh """
                        # Update deployment with new unique images
                        kubectl set image deployment/${K8S_DEPLOYMENT} \
                          backend=${BACKEND_IMAGE} \
                          frontend=${FRONTEND_IMAGE} \
                          -n ${K8S_NAMESPACE}

                        # Wait for pods to rollout
                        kubectl rollout status deployment/${K8S_DEPLOYMENT} -n ${K8S_NAMESPACE}
                    """
                }
            }
        }

        stage('Confirm Deployment') {
            steps {
                sh """
                    echo '✅ Current pods and images in namespace ${K8S_NAMESPACE}:'
                    kubectl get pods -n ${K8S_NAMESPACE} -o wide
                    kubectl get deployment ${K8S_DEPLOYMENT} -n ${K8S_NAMESPACE} -o=jsonpath="{.spec.template.spec.containers[*].image}"
                """
            }
        }

    }

    post {
        success {
            echo "✅ CI/CD Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}

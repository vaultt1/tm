pipeline {
    agent any

    environment {
        // Docker
        DOCKER_USER     = "someone15me"

        // K8s
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

        stage('Generate Image Tags') {
            steps {
                script {
                    // Jenkins build number + short Git SHA
                    GIT_SHA = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    BACKEND_TAG = "backend-${BUILD_NUMBER}-${GIT_SHA}"
                    FRONTEND_TAG = "frontend-${BUILD_NUMBER}-${GIT_SHA}"

                    env.BACKEND_IMAGE = "${DOCKER_USER}/dp:${BACKEND_TAG}"
                    env.FRONTEND_IMAGE = "${DOCKER_USER}/dp:${FRONTEND_TAG}"

                    echo "Backend Image: ${BACKEND_IMAGE}"
                    echo "Frontend Image: ${FRONTEND_IMAGE}"
                }
            }
        }

        stage('Build Backend Image') {
            steps {
                sh "docker build -t ${BACKEND_IMAGE} ./Backend"
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh "docker build -t ${FRONTEND_IMAGE} ./Frontend"
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([string(credentialsId: 'DOCKERHUB_TOKEN', variable: 'DOCKER_TOKEN')]) {
                    sh "echo $DOCKER_TOKEN | docker login -u ${DOCKER_USER} --password-stdin"
                }
            }
        }

        stage('Push Images') {
            steps {
                sh "docker push ${BACKEND_IMAGE}"
                sh "docker push ${FRONTEND_IMAGE}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'KUBECONFIG_FILE', variable: 'KUBECONFIG')]) {
                    sh """
                    kubectl set image deployment/${K8S_DEPLOYMENT} \
                        backend=${BACKEND_IMAGE} \
                        frontend=${FRONTEND_IMAGE} \
                        -n ${K8S_NAMESPACE}

                    kubectl rollout status deployment/${K8S_DEPLOYMENT} -n ${K8S_NAMESPACE}
                    """
                }
            }
        }

    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}

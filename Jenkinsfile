pipeline {
    agent any

    environment {
        DOCKER_USER     = "someone15me"
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

        stage('Set Image Tags') {
            steps {
                script {
                    // Use Jenkins build number or Git commit SHA
                    env.BUILD_TAG = "${env.BUILD_NUMBER}"  // or "git rev-parse --short HEAD"
                    env.BACKEND_IMAGE  = "someone15me/dp:backend-${env.BUILD_TAG}"
                    env.FRONTEND_IMAGE = "someone15me/dp:frontend-${env.BUILD_TAG}"
                    echo "Backend Image: ${env.BACKEND_IMAGE}"
                    echo "Frontend Image: ${env.FRONTEND_IMAGE}"
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

    triggers {
        // Poll SCM every minute
        pollSCM('H/1 * * * *')
    }

    post {
        success { echo "✅ Pipeline completed successfully!" }
        failure { echo "❌ Pipeline failed!" }
    }
}

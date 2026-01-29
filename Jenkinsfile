pipeline {
    agent any

    environment {
        BACKEND_IMAGE   = "someone15me/dp:backend"
        FRONTEND_IMAGE  = "someone15me/dp:frontend"
        DEPLOYMENT_NAME = "task-management-app"
        K8S_NAMESPACE   = "task-management"
        BUILD_TAG       = "${env.BUILD_NUMBER}-${GIT_COMMIT.substring(0,7)}"
        DC_DATA_DIR     = "${WORKSPACE}/dependency-check-data"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('OWASP Dependency-Check (SCA)') {
            steps {
                withCredentials([
                    string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')
                ]) {
                    sh """
                        mkdir -p dependency-check-report
                        dependency-check.sh \
                          --project "TaskManagementApp" \
                          --scan Backend \
                          --scan Frontend \
                          --format HTML \
                          --format JSON \
                          --out dependency-check-report \
                          --data ${DC_DATA_DIR} \
                          --nvdApiKey \$NVD_API_KEY \
                          --disableAutoUpdate \
                          --failOnCVSS 7 || true
                    """
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'dependency-check-report/*', fingerprint: true
                }
            }
        }

        stage('Trivy Filesystem Scan (Pre-Build)') {
            steps {
                sh """
                    trivy fs \
                      --severity HIGH,CRITICAL \
                      --exit-code 1 \
                      --no-progress \
                      .
                """
            }
        }

        stage('Build Docker Images') {
            steps {
                sh "docker build -t ${BACKEND_IMAGE}-${BUILD_TAG} ./Backend"
                sh "docker build -t ${FRONTEND_IMAGE}-${BUILD_TAG} ./Frontend"
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh """
                    trivy image \
                      --severity HIGH,CRITICAL \
                      --exit-code 1 \
                      --no-progress \
                      ${BACKEND_IMAGE}-${BUILD_TAG}

                    trivy image \
                      --severity HIGH,CRITICAL \
                      --exit-code 1 \
                      --no-progress \
                      ${FRONTEND_IMAGE}-${BUILD_TAG}
                """
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh """
                        echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                        docker push ${BACKEND_IMAGE}-${BUILD_TAG}
                        docker push ${FRONTEND_IMAGE}-${BUILD_TAG}
                        docker logout
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')
                ]) {
                    sh """
                        export KUBECONFIG=\$KUBECONFIG
                        kubectl get nodes
                        kubectl apply -f k8s/namespace.yaml
                        kubectl apply -f k8s/service.yaml
                        kubectl apply -f k8s/deployment.yaml -n ${K8S_NAMESPACE}
                        kubectl set image deployment/${DEPLOYMENT_NAME} \
                            backend=${BACKEND_IMAGE}-${BUILD_TAG} \
                            frontend=${FRONTEND_IMAGE}-${BUILD_TAG} \
                            -n ${K8S_NAMESPACE}
                        kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${K8S_NAMESPACE}
                    """
                }
            }
        }

        stage('Verification') {
            steps {
                sleep(time: 1, unit: 'MINUTES')
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')
                ]) {
                    sh """
                        export KUBECONFIG=\$KUBECONFIG
                        kubectl get all -n ${K8S_NAMESPACE}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Secure Deployment completed with BUILD_TAG=${BUILD_TAG}"
        }
        failure {
            echo "❌ Pipeline blocked due to security or deployment failure"
        }
    }
}

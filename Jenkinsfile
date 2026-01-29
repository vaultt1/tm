pipeline {
    agent any

    environment {
        BACKEND_IMAGE   = "someone15me/dp:backend"
        FRONTEND_IMAGE  = "someone15me/dp:frontend"
        DEPLOYMENT_NAME = "task-management-app"
        K8S_NAMESPACE   = "task-management"
        BUILD_TAG       = "${env.BUILD_NUMBER}-${GIT_COMMIT.substring(0,7)}"
        NVD_API_KEY     = credentials('nvd-api-key') // Store your NVD API key in Jenkins credentials
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Dependency Check (OWASP)') {
            steps {
                script {
                    sh """
                        # Install Dependency-Check CLI if not present
                        if ! command -v dependency-check.sh &> /dev/null; then
                            echo "Installing Dependency-Check CLI..."
                            curl -L -o dependency-check.zip https://github.com/jeremylong/DependencyCheck/releases/download/v8.4.1/dependency-check-8.4.1-release.zip
                            unzip dependency-check.zip -d dependency-check
                            chmod +x dependency-check/dependency-check/bin/dependency-check.sh
                        fi

                        # Run Dependency-Check for Backend
                        ./dependency-check/dependency-check/bin/dependency-check.sh \
                            --project "Backend" \
                            --scan ./Backend \
                            --out ./dependency-check-reports/backend \
                            --nvdApiKey ${NVD_API_KEY} \
                            --format ALL

                        # Run Dependency-Check for Frontend
                        ./dependency-check/dependency-check/bin/dependency-check.sh \
                            --project "Frontend" \
                            --scan ./Frontend \
                            --out ./dependency-check-reports/frontend \
                            --nvdApiKey ${NVD_API_KEY} \
                            --format ALL
                    """
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh "docker build -t ${BACKEND_IMAGE}-${BUILD_TAG} ./Backend"
                sh "docker build -t ${FRONTEND_IMAGE}-${BUILD_TAG} ./Frontend"
            }
        }

        stage('Trivy Scan Docker Images') {
            steps {
                script {
                    sh """
                        # Install Trivy if not present
                        if ! command -v trivy &> /dev/null; then
                            echo "Installing Trivy..."
                            curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh
                        fi

                        # Scan Backend Image
                        trivy image --exit-code 1 --severity HIGH,CRITICAL ${BACKEND_IMAGE}-${BUILD_TAG}

                        # Scan Frontend Image
                        trivy image --exit-code 1 --severity HIGH,CRITICAL ${FRONTEND_IMAGE}-${BUILD_TAG}
                    """
                }
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
                sh "ip a"
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
            echo "✅ Deployment completed with BUILD_TAG=${BUILD_TAG}"
            echo "Done"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}

pipeline {
    agent any

    environment {
        BACKEND_IMAGE   = "someone15me/dp:backend"
        FRONTEND_IMAGE  = "someone15me/dp:frontend"
        DEPLOYMENT_NAME = "task-management-app"
        K8S_NAMESPACE   = "task-management"
        BUILD_TAG       = "${env.BUILD_NUMBER}-${GIT_COMMIT.substring(0,7)}"
        NVD_API_KEY     = credentials('NVD_API_KEY')
    }

    options {
        timestamps()
        skipDefaultCheckout(false)
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Dependency Check (OWASP)') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    sh """
                        if [ ! -d "dependency-check" ]; then
                          echo "Installing Dependency-Check..."
                          curl -L -o dc.zip https://github.com/jeremylong/DependencyCheck/releases/download/v8.4.1/dependency-check-8.4.1-release.zip
                          unzip -q dc.zip
                          mv dependency-check-* dependency-check
                        fi

                        mkdir -p dependency-check-reports

                        ./dependency-check/bin/dependency-check.sh \
                          --project "Backend" \
                          --scan ./Backend \
                          --out dependency-check-reports/backend \
                          --format ALL \
                          --failOnCVSS 9 \
                          --nvdApiKey ${NVD_API_KEY}

                        ./dependency-check/bin/dependency-check.sh \
                          --project "Frontend" \
                          --scan ./Frontend \
                          --out dependency-check-reports/frontend \
                          --format ALL \
                          --failOnCVSS 9 \
                          --nvdApiKey ${NVD_API_KEY}
                    """
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh """
                    docker build -t ${BACKEND_IMAGE}-${BUILD_TAG} ./Backend
                    docker build -t ${FRONTEND_IMAGE}-${BUILD_TAG} ./Frontend
                """
            }
        }

        stage('Trivy Image Scan') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    sh """
                        if ! command -v trivy >/dev/null 2>&1; then
                          echo "Installing Trivy..."
                          curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh
                          sudo mv trivy /usr/local/bin/
                        fi

                        trivy image \
                          --severity CRITICAL \
                          --ignore-unfixed \
                          ${BACKEND_IMAGE}-${BUILD_TAG}

                        trivy image \
                          --severity CRITICAL \
                          --ignore-unfixed \
                          ${FRONTEND_IMAGE}-${BUILD_TAG}
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
            when {
                expression { currentBuild.result == 'SUCCESS' }
            }
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')
                ]) {
                    sh """
                        export KUBECONFIG=\$KUBECONFIG

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
            echo "✅ Deployment completed successfully"
            echo "BUILD_TAG=${BUILD_TAG}"
        }
        unstable {
            echo "⚠️ Deployment completed with security warnings"
        }
        failure {
            echo "❌ Pipeline failed (build, push, or deploy issue)"
        }
    }
}


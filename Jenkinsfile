pipeline {
    agent any

    environment {
        NVD_API_KEY = credentials('NVD_API_KEY') // Jenkins credential for OWASP NVD
        DOCKER_REGISTRY = "your-docker-registry"
        APP_NAME = "taskmanagementapp"
        KUBE_CONTEXT = "your-kube-context"
        KUBE_NAMESPACE = "default"
    }

    options {
        skipDefaultCheckout()
        timestamps()
        ansiColor('xterm')
    }

    stages {
        // -----------------------------
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        // -----------------------------
        stage('Install Dependencies') {
            steps {
                dir('Backend') { sh 'npm install' }
                dir('Frontend') { sh 'npm install' }
            }
        }

        // -----------------------------
        stage('OWASP Dependency-Check (SCA)') {
            steps {
                script {
                    sh '''
                    mkdir -p dependency-check-report
                    dependency-check.sh \
                        --project ${APP_NAME} \
                        --scan Backend --scan Frontend \
                        --format HTML --format JSON \
                        --out dependency-check-report \
                        --data ${WORKSPACE}/dependency-check-data \
                        --nvdApiKey $NVD_API_KEY \
                        --noupdate || true
                    '''
                }
            }
        }

        stage('Archive SCA Reports') {
            steps {
                archiveArtifacts artifacts: 'dependency-check-report/*', allowEmptyArchive: true
            }
        }

        // -----------------------------
        stage('Trivy Filesystem Scan (Pre-Build)') {
            steps {
                script {
                    sh '''
                    mkdir -p trivy-report
                    trivy fs --severity HIGH,CRITICAL --exit-code 1 \
                        --format json -o trivy-report/fs-report.json .
                    '''
                }
            }
        }

        stage('Archive Trivy Reports') {
            steps {
                archiveArtifacts artifacts: 'trivy-report/*', allowEmptyArchive: true
            }
        }

        // -----------------------------
        stage('Build Docker Images') {
            steps {
                script {
                    sh """
                    docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:backend Backend
                    docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:frontend Frontend
                    """
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                script {
                    sh """
                    mkdir -p trivy-image-report
                    trivy image --severity HIGH,CRITICAL --exit-code 1 \
                        --format json -o trivy-image-report/backend.json ${DOCKER_REGISTRY}/${APP_NAME}:backend
                    trivy image --severity HIGH,CRITICAL --exit-code 1 \
                        --format json -o trivy-image-report/frontend.json ${DOCKER_REGISTRY}/${APP_NAME}:frontend
                    """
                }
            }
        }

        stage('Archive Image Scan Reports') {
            steps {
                archiveArtifacts artifacts: 'trivy-image-report/*', allowEmptyArchive: true
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    sh """
                    docker push ${DOCKER_REGISTRY}/${APP_NAME}:backend
                    docker push ${DOCKER_REGISTRY}/${APP_NAME}:frontend
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    kubectl config use-context ${KUBE_CONTEXT}
                    kubectl apply -f k8s/deployment.yaml -n ${KUBE_NAMESPACE}
                    kubectl rollout status deployment/${APP_NAME}-backend -n ${KUBE_NAMESPACE}
                    kubectl rollout status deployment/${APP_NAME}-frontend -n ${KUBE_NAMESPACE}
                    """
                }
            }
        }

        stage('Post-Deployment Verification') {
            steps {
                script {
                    echo "✅ Deployment complete. Running verification..."
                    sh 'kubectl get pods -n ${KUBE_NAMESPACE}'
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check reports for details."
        }
        always {
            cleanWs()
        }
    }
}

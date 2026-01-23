pipeline {
    agent any

    environment {
        DOCKER_USER    = "someone15me"
        K8S_NAMESPACE  = "task-management"
        K8S_DEPLOYMENT = "task-management-app"
    }

    triggers {
        pollSCM('H/2 * * * *') // Example: Polls every 5 minutes
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
                    // Use Jenkins build number for uniqueness
                    BUILD_TAG = "${env.BUILD_NUMBER}"

                    // Optional: include short git commit SHA for traceability
                    GIT_SHA = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()

                    BACKEND_IMAGE  = "someone15me/dp:backend-${BUILD_TAG}-${GIT_SHA}"
                    FRONTEND_IMAGE = "someone15me/dp:frontend-${BUILD_TAG}-${GIT_SHA}"

                    echo "➡️ Backend Image: ${BACKEND_IMAGE}"
                    echo "➡️ Frontend Image: ${FRONTEND_IMAGE}"
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

        stage('Push Images to DockerHub') {
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
            echo "✅ CI/CD Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}

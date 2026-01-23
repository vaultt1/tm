pipeline {
    agent any

    environment {
        // Docker
        DOCKER_USER     = "someone15me"
        BACKEND_IMAGE   = "someone15me/dp:backend-latest"
        FRONTEND_IMAGE  = "someone15me/dp:frontend-latest"

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

        stage('Build Backend Image') {
            steps {
                sh '''
                  docker build -t ${BACKEND_IMAGE} ./backend
                '''
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh '''
                  docker build -t ${FRONTEND_IMAGE} ./frontend
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([string(credentialsId: 'DOCKERHUB_TOKEN', variable: 'DOCKER_TOKEN')]) {
                    sh '''
                      echo $DOCKER_TOKEN | docker login -u ${DOCKER_USER} --password-stdin
                    '''
                }
            }
        }

        stage('Push Images to DockerHub') {
            steps {
                sh '''
                  docker push ${BACKEND_IMAGE}
                  docker push ${FRONTEND_IMAGE}
                '''
            }
        }

        stage('restart service') {
            steps {
                withCredentials([file(credentialsId: 'KUBECONFIG_FILE', variable: 'KUBECONFIG')]) {
                    sh 'kubectl set image deployment/task-management-app backend=someone15me/dp:backend-latest frontend=someone15me/dp:frontend-latest -n task-management'
                    sh 'kubectl rollout status deployment/task-management-app -n task-management'
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

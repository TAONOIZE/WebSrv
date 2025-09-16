pipeline {
    agent any

    environment {
        GITHUB_REPO = 'https://github.com/TAONOIZE/WebSrv.git'
        BRANCH = 'main'
        DOCKER_IMAGE = "nktdkr23/php-apache"
        K8S_NAMESPACE = "utcc-it-dev"
        DEPLOYMENT_NAME = "web-deployment"
        CREDENTIALS_ID = "ecb3549b-e779-4e49-981e-adde049fd5e3"
        CREDENTIALS_KUBEID = "fb178c28-43bb-4f81-8353-9aa4c0415f93"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${BRANCH}", url: "${GITHUB_REPO}"
            }
        }

        stage('Build & Test') {
            steps {
                sh 'echo "Building project..."'
                // Example for Node.js:
                // sh 'npm install && npm test'
                // Example for Java:
                // sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]){
                    sh """
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'nkp-kube-dev']) {
                    sh """
                    # Update Kubernetes deployment to use the new image
                    kubectl set image deployment/${DEPLOYMENT_NAME} ${DEPLOYMENT_NAME}=${DOCKER_IMAGE}:${BUILD_NUMBER} -n ${K8S_NAMESPACE}
                    # Wait for rollout to complete
                    kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${K8S_NAMESPACE}
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs.'
        }
    }
}

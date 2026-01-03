pipeline {

    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'dockerhub-creds'
        DOCKER_REGISTRY = 'rabtab'
        IMAGE_TAG = "${BUILD_NUMBER}"
        GIT_COMMIT_SHORT = "${GIT_COMMIT[0..6]}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/rab-tab/movie-booking-kafka-saga.git'
            }
        }

        stage('Build & Unit Tests') {
            steps {
                sh 'mvn -T 1C clean test'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn -T 1C package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            parallel {

                stage('Booking Service') {
                    steps {
                        buildAndPush(
                            "booking-service",
                            "booking-service/Dockerfile"
                        )
                    }
                }

                stage('Seat Inventory Service') {
                    steps {
                        buildAndPush(
                            "seat-inventory-service",
                            "seat-inventory-service/Dockerfile"
                        )
                    }
                }

                stage('Payment Service') {
                    steps {
                        buildAndPush(
                            "payment-service",
                            "payment-service/Dockerfile"
                        )
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}

def buildAndPush(serviceName, dockerfilePath) {
    withCredentials([
        usernamePassword(
            credentialsId: DOCKER_HUB_CREDENTIALS,
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )
    ]) {
        sh """
            echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin

            docker build \
              -f ${dockerfilePath} \
              -t ${DOCKER_REGISTRY}/${serviceName}:${IMAGE_TAG} \
              -t ${DOCKER_REGISTRY}/${serviceName}:${GIT_COMMIT_SHORT} \
              ${serviceName}

            docker push ${DOCKER_REGISTRY}/${serviceName}:${IMAGE_TAG}
            docker push ${DOCKER_REGISTRY}/${serviceName}:${GIT_COMMIT_SHORT}
        """
    }
}

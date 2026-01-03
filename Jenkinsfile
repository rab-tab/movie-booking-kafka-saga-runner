pipeline {
    agent any

    environment {
        IMAGE_TAG = 'latest' // or injected from upstream pipeline
    }

    stages {

        stage('Pull Images') {
            steps {
                sh '''
                  docker pull rabtab/booking-service:$IMAGE_TAG
                  docker pull rabtab/seat-inventory-service:$IMAGE_TAG
                  docker pull rabtab/payment-service:$IMAGE_TAG
                '''
            }
        }
        stage('Cleanup Previous Stack') {
          steps {
            sh 'docker-compose down -v --remove-orphans || true'
          }
        }
        stage('Start Stack') {
            steps {
                sh 'docker-compose up -d'
            }
        }

        stage('Wait for Services') {
            steps {
                sh '''
                  sleep 30
                  docker ps
                '''
            }
        }

        stage('Smoke Tests') {
            steps {
                sh '''
                  curl -f http://localhost:9191/actuator/health
                  curl -f http://localhost:9292/actuator/health
                  curl -f http://localhost:9393/actuator/health
                '''
            }
        }
    }

    post {
        always {
           sh '''
                     docker-compose down -v --remove-orphans || true
                   '''
        }
    }
}

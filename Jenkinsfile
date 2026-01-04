pipeline {
    agent any

    environment {
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Docker Cleanup') {
          steps {
            sh '''
              docker-compose down -v --remove-orphans || true
              docker rm -f jaeger || true
            '''
          }
        }

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
                  wait_for_health() {
                    local url=$1
                    local name=$2
                    local retries=30
                    local wait=5

                    echo "⏳ Waiting for $name to be UP..."

                    for i in $(seq 1 $retries); do
                      if curl -fs "$url" | grep -q '"status":"UP"'; then
                        echo "✅ $name is UP"
                        return 0
                      fi

                      echo "⏱ $name not ready yet ($i/$retries)"
                      sleep $wait
                    done

                    echo "❌ $name failed to start"
                    exit 1
                  }

                  wait_for_health http://localhost:9191/actuator/health booking-service
                  wait_for_health http://localhost:9292/actuator/health seat-inventory-service
                  wait_for_health http://localhost:9393/actuator/health payment-service
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
            sh 'docker-compose down -v --remove-orphans || true'
        }
    }
}
pipeline {
    agent any

    environment {
        IMAGE_TAG = 'latest'
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
                  wait_for_health() {
                    local url=$1
                    local name=$2
                    local retries=30
                    local wait=5

                    echo "⏳ Waiting for $name to be UP..."

                    for i in $(seq 1 $retries); do
                      if curl -fs "$url" | grep -q '"status":"UP"'; then
                        echo "✅ $name is UP"
                        return 0
                      fi

                      echo "⏱ $name not ready yet ($i/$retries)"
                      sleep $wait
                    done

                    echo "❌ $name failed to start"
                    exit 1
                  }

                  wait_for_health http://localhost:9191/actuator/health booking-service
                  wait_for_health http://localhost:9292/actuator/health seat-inventory-service
                  wait_for_health http://localhost:9393/actuator/health payment-service
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
            sh 'docker-compose down -v --remove-orphans || true'
        }
    }
}

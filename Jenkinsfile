version: "3.8"

networks:
  movie-booking-net:
    driver: bridge

services:

  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    container_name: zookeeper
    networks:
      - movie-booking-net
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    healthcheck:
      test: ["CMD", "bash", "-c", "echo ruok | nc localhost 2181 | grep imok"]
      interval: 10s
      timeout: 5s
      retries: 10

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka
    networks:
      - movie-booking-net
    ports:
      - "9092:9092"
    depends_on:
      zookeeper:
        condition: service_healthy
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181

      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092

      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
    healthcheck:
      test: ["CMD", "bash", "-c", "kafka-topics --bootstrap-server localhost:9092 --list"]
      interval: 10s
      timeout: 5s
      retries: 10

  booking-service:
    image: rabtab/booking-service:${IMAGE_TAG:-latest}
    container_name: booking-service
    networks:
      - movie-booking-net
    ports:
      - "9191:9191"
    depends_on:
      kafka:
        condition: service_healthy
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SERVER_PORT: 9191
      SPRING_KAFKA_BOOTSTRAP_SERVERS: kafka:9092
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9191/actuator/health"]
      interval: 10s
      timeout: 5s
      retries: 15

  seat-inventory-service:
    image: rabtab/seat-inventory-service:${IMAGE_TAG:-latest}
    container_name: seat-inventory-service
    networks:
      - movie-booking-net
    ports:
      - "9292:9292"
    depends_on:
      kafka:
        condition: service_healthy
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SERVER_PORT: 9292
      SPRING_KAFKA_BOOTSTRAP_SERVERS: kafka:9092
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9292/actuator/health"]
      interval: 10s
      timeout: 5s
      retries: 15

  payment-service:
    image: rabtab/payment-service:${IMAGE_TAG:-latest}
    container_name: payment-service
    networks:
      - movie-booking-net
    ports:
      - "9393:9393"
    depends_on:
      kafka:
        condition: service_healthy
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SERVER_PORT: 9393
      SPRING_KAFKA_BOOTSTRAP_SERVERS: kafka:9092
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9393/actuator/health"]
      interval: 10s
      timeout: 5s
      retries: 15

# Event Driven Lab

Event-driven architecture lab built with Spring Boot and RabbitMQ.

This project includes two microservices:

- Producer: exposes a REST endpoint and publishes messages to RabbitMQ.
- Consumer: listens to the queue and processes received messages.

## Architecture

- Broker: RabbitMQ (`rabbitmq:management`)
- Producer: `producer-service`
- Consumer: `consumer-service`
- Docker network: `event_network`

Flow:

1. A client sends a `POST` request to the Producer.
2. The Producer publishes to exchange `messages.exchange` using routing key `messages.routingkey`.
3. RabbitMQ routes the message to queue `messages.queue`.
4. The Consumer receives and processes the message.

## Project Structure

```text
event-driven-lab/
├── docker-compose.yml
├── .gitignore
├── producer-service/
│   ├── Dockerfile
│   ├── pom.xml
│   └── src/main/
└── consumer-service/
		├── Dockerfile
		├── pom.xml
		└── src/main/
```

## Implemented Configuration

### Producer (`producer-service`)

- Java 17, Spring Boot, Spring Web, Spring AMQP.
- Endpoint:

```http
POST /api/messages/send?message=<text>
```

- RabbitMQ properties:

```properties
spring.rabbitmq.host=rabbitmq
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest

app.rabbitmq.exchange=messages.exchange
app.rabbitmq.queue=messages.queue
app.rabbitmq.routingkey=messages.routingkey
```

- AMQP setup:
  - Durable queue: `messages.queue`
  - DirectExchange: `messages.exchange`
  - Binding by `messages.routingkey`

### Consumer (`consumer-service`)

- Java 17, Spring Boot, Spring AMQP.
- Listener:

```java
@RabbitListener(queues = "${app.rabbitmq.queue}")
public void receiveMessage(String message) { ... }
```

- RabbitMQ properties:

```properties
spring.rabbitmq.host=rabbitmq
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest

app.rabbitmq.queue=messages.queue
```

## Dockerfiles

- Producer base image: `eclipse-temurin:17-jdk-jammy`
- Consumer base image: `eclipse-temurin:17-jre-jammy`

Both services run with:

```bash
java -jar app.jar
```

## Docker Compose

`docker-compose.yml` includes:

- RabbitMQ Management (ports `5672` and `15672`)
- Producer (`sergiobejarano/producer-service`)
- Consumer (`sergiobejarano/consumer-service`)

## Docker Hub Publishing

Published images:

- `sergiobejarano/producer-service:latest`
- `sergiobejarano/consumer-service:latest`

## How to Run

### Option 1: Start with Docker Compose

With Docker Compose V1:

```bash
docker-compose up -d
docker-compose ps
```

With Docker Compose V2:

```bash
docker compose up -d
docker compose ps
```

### Option 2: Test Message Publishing

```bash
curl -X POST "http://localhost:8080/api/messages/send?message=HelloWorld"
```

Expected response (current implementation text):

```text
Mensaje 'HelloWorld' enviado!
```

Check consumer logs:

```bash
docker-compose logs consumer
# or
docker compose logs consumer
```

Expected output (similar, current implementation text):

```text
Mensaje recibido: 'HelloWorld'
>>> Mensaje Procesado: HelloWorld
```

## RabbitMQ UI

- URL: `http://localhost:15672`
- Username: `guest`
- Password: `guest`

In the `Queues` tab you can inspect `messages.queue`.

## Local Build (Maven)

Producer:

```bash
cd producer-service
mvn package
```

Consumer:

```bash
cd consumer-service
mvn package
```

## Notes

- Opening `/api/messages/send` directly in a browser may return `405 Method Not Allowed`, because the endpoint accepts only `POST` requests.
- Depending on the environment, either `docker-compose` (V1) or `docker compose` (V2) may be available. Use the command supported by the current environment.

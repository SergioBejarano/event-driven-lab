# Event-Driven Lab

Event-driven architecture lab built with **Spring Boot** and **RabbitMQ**.

This project includes two microservices:

- **Producer:** exposes a REST endpoint and publishes messages to RabbitMQ.
- **Consumer:** listens to the queue and processes received messages.

---

## Architecture

| Component      | Detail                           |
| -------------- | -------------------------------- |
| Broker         | RabbitMQ (`rabbitmq:management`) |
| Producer       | `producer-service`               |
| Consumer       | `consumer-service`               |
| Docker network | `event_network`                  |

**Message Flow:**

1. A client sends a `POST` request to the Producer.
2. The Producer publishes to exchange `messages.exchange` using routing key `messages.routingkey`.
3. RabbitMQ routes the message to queue `messages.queue`.
4. The Consumer receives and processes the message.

---

## Project Structure

```
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

---

## Configuration

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

---

## Dockerfiles

- Producer base image: `eclipse-temurin:17-jdk-jammy`
- Consumer base image: `eclipse-temurin:17-jre-jammy`

Both services are started with:

```bash
java -jar app.jar
```

<img src="https://github.com/user-attachments/assets/6c4ed09f-d746-4ff6-bb0a-8ea79c876364" width="800" alt="Producer Dockerfile" />

<img src="https://github.com/user-attachments/assets/7914c52b-87d3-448a-a4c4-c669daeee2f5" width="800" alt="Consumer Dockerfile" />

---

## Docker Compose

`docker-compose.yml` includes:

- RabbitMQ Management (ports `5672` and `15672`)
- Producer (`sergiobejarano/producer-service`)
- Consumer (`sergiobejarano/consumer-service`)

<img src="https://github.com/user-attachments/assets/ded04208-f52f-4882-830a-617a319e69ab" width="800" alt="docker-compose.yml" />

---

## Docker Hub

Published images:

- `sergiobejarano/producer-service:latest`
- `sergiobejarano/consumer-service:latest`

---

## Running on Killercoda

[Killercoda](https://killercoda.com) is a free platform with Docker and Git pre-installed, accessible from the browser. It allows exposing ports via the **Traffic / Port** menu.

### Steps

**1. Open the playground**

Access [Killercoda Ubuntu Playground](https://killercoda.com/playgrounds/scenario/ubuntu) and sign in with a GitHub or Google account.

**2. Clone the repository**

```bash
git clone https://github.com/SergioBejarano/event-driven-lab.git
cd event-driven-lab/
```

**3. Start the services**

```bash
docker-compose up -d
```

> Wait ~30 seconds for RabbitMQ to be fully ready before sending messages.

**4. Verify containers are running**

```bash
docker-compose ps
```

**5. Send a message**

```bash
curl -X POST "http://localhost:8080/api/messages/send?message=HolaDesdeKillercoda"
```

Expected: `Mensaje 'HolaDesdeKillercoda' enviado!`

<img src="https://github.com/user-attachments/assets/39338654-d7a2-4f99-9d2f-2f0b5f2c47d6" width="800" alt="curl send message Killercoda" />

**6. Check the consumer**

```bash
docker-compose logs consumer
```

Expected:

```
Mensaje recibido: 'HolaDesdeKillercoda'
>>> Mensaje Procesado: HolaDesdeKillercoda
```

<img src="https://github.com/user-attachments/assets/6965b67d-1e43-4f39-90b4-dcbc3250bc67" width="800" alt="consumer logs Killercoda" />

**7. Access services from the browser**

Click the **Traffic / Port** button (top-right corner of the Killercoda terminal):

- Port `8080` → Producer API
- Port `15672` → RabbitMQ Management UI
  <img src="https://github.com/user-attachments/assets/2369164d-d02e-412d-adb1-777bfa8a4579" width="800" alt="docker-compose ps en Killercoda" />

Sign in with `guest` / `guest`. Navigate to **Queues** → `messages.queue`.
<img src="https://github.com/user-attachments/assets/e9638a14-ba2e-4efa-b0bc-95759897be6f" width="800" alt="RabbitMQ UI Killercoda" />

<img src="https://github.com/user-attachments/assets/4a193522-cdf6-475b-8776-40f6744bc6bc" width="800" alt="messages.queue detail view" />

### Troubleshooting

> **405 Method Not Allowed** when opening the producer URL in the browser — this is expected. The endpoint only accepts `POST` requests. Use `curl` or a tool like [Hoppscotch](https://hoppscotch.io) (set method to `POST`).

> **Network Error in Hoppscotch** — switch the Interceptor mode from _Browser_ to _Proxy_ to avoid CORS blocking.

> **Killercoda session limit** — public URLs are only valid during the active session (max 1 hour on the free plan). Images must be published to Docker Hub before starting.

---

## Local Build (Maven)

**Producer:**

```bash
cd producer-service
mvn package
```

**Consumer:**

```bash
cd consumer-service
mvn package
```

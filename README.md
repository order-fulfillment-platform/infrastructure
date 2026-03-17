[![LinkedIn][linkedin-shield]][linkedin-url]

<br />
<div align="center">
<h3 align="center">Order Fulfillment Platform — Infrastructure</h3>

  <p align="center">
    Docker Compose infrastructure configuration for the Order Fulfillment Platform, an event-driven microservices system built with Spring Boot 3, Apache Kafka and PostgreSQL.
    <br />
    <br />
    <a href="https://github.com/order-fulfillment-platform">View Organization</a>
  </p>
</div>

<details>
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#about-the-project">About The Project</a></li>
    <li><a href="#built-with">Built With</a></li>
    <li><a href="#architecture">Architecture</a></li>
    <li><a href="#services">Services</a></li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#troubleshooting">Troubleshooting</a></li>
    <li><a href="#contact">Contact</a></li>
  </ol>
</details>

## About The Project

The Order Fulfillment Platform is a distributed system that demonstrates modern microservices architecture patterns including event-driven communication, the outbox pattern, idempotency, and resilience.

The platform handles the full order lifecycle: from order creation, through stock reservation and payment authorization, to customer notification — all via asynchronous Kafka events.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Built With

[![Spring Boot][springboot-shield]][springboot-url]
[![Apache Kafka][kafka-shield]][kafka-url]
[![PostgreSQL][postgres-shield]][postgres-url]
[![Docker][docker-shield]][docker-url]
[![Java][java-shield]][java-url]

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Architecture
```
                        ┌─────────────────┐
                        │   API Gateway   │
                        │     :8080       │
                        └────────┬────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
     ┌────────▼────────┐ ┌───────▼───────┐ ┌────────▼──────┐
     │  Order Service  │ │   Inventory   │ │    Payment    │
     │    :8081        │ │   Service     │ │    Service    │
     │                 │ │   :8082       │ │    :8083      │
     └────────┬────────┘ └───────┬───────┘ └────────┬──────┘
              │                  │                  │
              └──────────────────┼──────────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │      Apache Kafka       │
                    │        :9092            │
                    └────────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │  Notification Service   │
                    │        :8084            │
                    └─────────────────────────┘
```

### Event Flow
```
POST /api/v1/orders
    → order-service        emits  ORDER_CREATED
    → inventory-service    emits  STOCK_RESERVED / STOCK_REJECTED
    → payment-service      emits  PAYMENT_AUTHORIZED / PAYMENT_FAILED
    → notification-service logs   email notification
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Services

### Application Services

| Service | Port | Description |
|---|---|---|
| api-gateway | 8080 | Single entry point for all clients |
| order-service | 8081 | Order creation and lifecycle management |
| inventory-service | 8082 | Stock reservation |
| payment-service | 8083 | Payment authorization (mock) |
| notification-service | 8084 | Email notifications (mock) |

### Infrastructure Services

| Service | Port | Description |
|---|---|---|
| order-postgres | 5432 | PostgreSQL for order-service |
| inventory-postgres | 5433 | PostgreSQL for inventory-service |
| payment-postgres | 5434 | PostgreSQL for payment-service |
| kafka | 9092 | Apache Kafka message broker |
| zookeeper | 2181 | Kafka coordination service |

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Getting Started

### Prerequisites

- Docker 24+
- Docker Compose 2+
- Java 21
- Maven 3.9+

### Installation

1. Clone all repositories
```bash
git clone https://github.com/OrderFulfillmentPlatform/infrastructure
git clone https://github.com/OrderFulfillmentPlatform/order-service
git clone https://github.com/OrderFulfillmentPlatform/inventory-service
git clone https://github.com/OrderFulfillmentPlatform/payment-service
git clone https://github.com/OrderFulfillmentPlatform/notification-service
git clone https://github.com/OrderFulfillmentPlatform/api-gateway
```

2. Build all services — run in each service directory
```bash
mvn package -DskipTests
```

3. Start the platform — run from the `infrastructure` directory
```bash
docker-compose up -d --build
```

4. Verify all services are running
```bash
docker-compose ps
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Usage

### Create an order
```bash
curl -X POST http://localhost:8080/api/v1/orders \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": "a3f8c2d1-1234-5678-abcd-ef0123456789",
    "items": [
      {
        "productId": "b7e9f3a2-1234-5678-abcd-ef0123456789",
        "quantity": 2,
        "unitPrice": 29.99
      }
    ]
  }'
```

### Check payment status
```bash
curl http://localhost:8080/api/v1/payments/order/{orderId}
```

### Stop the platform
```bash
docker-compose down
```

To also remove all volumes:
```bash
docker-compose down -v
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Troubleshooting

**Services fail to start** — Some services may take up to 60 seconds to start due to JVM warmup. Wait for all containers to be healthy before testing.

**Kafka connection issues** — Restart the Kafka container:
```bash
docker-compose restart kafka
```

**View service logs:**
```bash
docker logs <service-name>
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Contact

Eros Burelli — [LinkedIn](https://www.linkedin.com/in/eros-burelli-a458b1145/)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- MARKDOWN LINKS & IMAGES -->
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/eros-burelli-a458b1145/
[springboot-shield]: https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=white
[springboot-url]: https://spring.io/projects/spring-boot
[kafka-shield]: https://img.shields.io/badge/Apache_Kafka-231F20?style=for-the-badge&logo=apache-kafka&logoColor=white
[kafka-url]: https://kafka.apache.org/
[postgres-shield]: https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white
[postgres-url]: https://www.postgresql.org/
[docker-shield]: https://img.shields.io/badge/Docker-2CA5E0?style=for-the-badge&logo=docker&logoColor=white
[docker-url]: https://www.docker.com/
[java-shield]: https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white
[java-url]: https://www.java.com/
[![LinkedIn][linkedin-shield]][linkedin-url]

<br />
<div align="center">
<h3 align="center">Order Fulfillment Platform — Infrastructure</h3>

  <p align="center">
    Docker Compose and Kubernetes infrastructure configuration for the Order Fulfillment Platform, an event-driven microservices system built with Spring Boot 3, Apache Kafka and PostgreSQL.
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
        <li><a href="#run-with-docker-compose">Run with Docker Compose</a></li>
        <li><a href="#run-with-kubernetes">Run with Kubernetes</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#troubleshooting">Troubleshooting</a></li>
    <li><a href="#contact">Contact</a></li>
  </ol>
</details>

## About The Project

The Order Fulfillment Platform is a distributed system that demonstrates modern microservices architecture patterns including event-driven communication, the Outbox Pattern, idempotency, and resilience.

The platform handles the full order lifecycle: from order creation, through stock reservation and payment authorization, to customer notification — all via asynchronous Kafka events.

This repository contains two deployment options: Docker Compose for local development and Kubernetes manifests for a production-like environment using Minikube.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Built With

[![Spring Boot][springboot-shield]][springboot-url]
[![Apache Kafka][kafka-shield]][kafka-url]
[![PostgreSQL][postgres-shield]][postgres-url]
[![Docker][docker-shield]][docker-url]
[![Kubernetes][kubernetes-shield]][kubernetes-url]
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
     ┌────────▼────────┐ ┌───────▼───────┐ ┌───────▼───────┐
     │  Order Service  │ │   Inventory   │ │    Payment    │
     │    :8081        │ │   Service     │ │    Service    │
     │                 │ │   :8082       │ │    :8083      │
     └────────┬────────┘ └───────┬───────┘ └───────┬───────┘
              │                  │                  │
              └──────────────────┼──────────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │      Apache Kafka        │
                    │        :9092             │
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
| kafka | 9092 | Apache Kafka message broker (KRaft mode) |

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Getting Started

### Prerequisites

- Docker 24+
- Docker Compose 2+
- Java 21
- Maven 3.9+
- Minikube (for Kubernetes deployment)
- kubectl (for Kubernetes deployment)

---

### Run with Docker Compose

**1. Clone all repositories**

```bash
git clone https://github.com/order-fulfillment-platform/infrastructure
git clone https://github.com/order-fulfillment-platform/order-service
git clone https://github.com/order-fulfillment-platform/inventory-service
git clone https://github.com/order-fulfillment-platform/payment-service
git clone https://github.com/order-fulfillment-platform/notification-service
git clone https://github.com/order-fulfillment-platform/api-gateway
```

**2. Create the `.env` file from the example**

```bash
cp .env.example .env
```

**3. Start the platform**

```bash
docker-compose up -d --build
```

**4. Verify all services are running**

```bash
docker-compose ps
```

---

### Run with Kubernetes

**1. Start Minikube**

```bash
minikube start
```

**2. Configure Docker to use Minikube's registry**

Linux/macOS:
```bash
eval $(minikube docker-env)
```

Windows PowerShell:
```powershell
& minikube -p minikube docker-env --shell powershell | Invoke-Expression
```

**3. Build all Docker images inside Minikube**

```bash
docker build -t order-service:1.0.0 ./order-service
docker build -t inventory-service:1.0.0 ./inventory-service
docker build -t payment-service:1.0.0 ./payment-service
docker build -t notification-service:1.0.0 ./notification-service
docker build -t api-gateway:1.0.0 ./api-gateway
```

**4. Create the namespace and deploy all resources**

```bash
kubectl create namespace ofp
kubectl apply -k k8s/
```

**5. Verify all pods are running**

```bash
kubectl get pods -n ofp
```

**6. Get the API Gateway URL**

```bash
minikube service api-gateway -n ofp --url
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

Replace `localhost:8080` with the Minikube URL when using Kubernetes.

### Check payment status

```bash
curl http://localhost:8080/api/v1/payments/order/{orderId}
```

### Stop the platform

Docker Compose:
```bash
docker-compose down
```

To also remove all volumes:
```bash
docker-compose down -v
```

Kubernetes:
```bash
kubectl delete namespace ofp
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Troubleshooting

**Services fail to start** — Some services may take up to 60 seconds to start due to JVM warmup. Wait for all containers to be healthy before testing.

**Kafka connection issues** — Restart the Kafka container:

Docker Compose:
```bash
docker-compose restart kafka
```

Kubernetes:
```bash
kubectl rollout restart deployment kafka -n ofp
```

**View service logs:**

Docker Compose:
```bash
docker logs <service-name>
```

Kubernetes:
```bash
kubectl logs -n ofp <pod-name>
```

**Pods stuck in CrashLoopBackOff:**
```bash
kubectl describe pod -n ofp <pod-name>
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Contact

Eros Burelli — [LinkedIn](https://www.linkedin.com/in/eros-burelli-a458b1145/)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- MARKDOWN LINKS -->
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
[kubernetes-shield]: https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white
[kubernetes-url]: https://kubernetes.io/
[java-shield]: https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white
[java-url]: https://www.java.com/

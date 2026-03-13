# CST8915 Full-Stack Cloud-Native Development
## Lab 5 — Containerizing the Algonquin Pet Store (Microservices with Docker)

---

## Student Information

| Field | Details |
|---|---|
| **Student Name** | Divyang Lodariya |
| **Student ID** | 041267824 |
| **Course** | CST8915 Full-Stack Cloud-Native Development |
| **Semester** | Winter 2026 |
| **GitHub** | [Divyang2599](https://github.com/Divyang2599) |

---

## Demo Video

▶️ **[Watch the Lab 5 Demo on YouTube](https://www.youtube.com/watch?v=REPLACE_WITH_YOUR_LINK)**

> The demo covers: building Docker images, running all services with Docker Compose, and placing a live order on the Algonquin Pet Store running on an Azure VM.

---

## Project Overview

This lab containerizes the **Algonquin Pet Store** — a microservices-based e-commerce application. Each service runs in its own Docker container, and all services are orchestrated using Docker Compose.

### Architecture

```
Browser (User)
      │
      ▼
┌─────────────┐
│ store-front │  Vue.js — Port 80
│  (Nginx)    │
└──────┬──────┘
       │
  ┌────┴────┐
  ▼         ▼
┌─────────┐ ┌──────────────┐
│  order  │ │   product    │
│ service │ │   service    │
│(Node.js)│ │  (Python)    │
│Port 3000│ │  Port 3030   │
└────┬────┘ └──────────────┘
     │
     ▼
┌──────────┐
│ RabbitMQ │  Message Queue — Port 5672
└──────────┘
```

---

## Repositories

| Service | GitHub Repository |
|---|---|
| **order-service** | [Divyang2599/order-service-L4](https://github.com/Divyang2599/order-service-L4) |
| **product-service** | [Divyang2599/product-service-python-L4](https://github.com/Divyang2599/product-service-python-L4) |
| **store-front** | [Divyang2599/store-front-L4](https://github.com/Divyang2599/store-front-L4) |

---

## Docker Hub Images

| Service | Docker Hub Image |
|---|---|
| **order-service** | [divyang25/order-service:latest](https://hub.docker.com/r/divyang25/order-service) |
| **product-service** | [divyang25/product-service:latest](https://hub.docker.com/r/divyang25/product-service) |
| **store-front** | [divyang25/store-front:latest](https://hub.docker.com/r/divyang25/store-front) |

---

## Docker Compose File

The final `docker-compose.yml` uses images pulled directly from Docker Hub:

```yaml
version: '3'

services:
  rabbitmq:
    image: "rabbitmq:3-management"
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      - RABBITMQ_DEFAULT_USER=myuser
      - RABBITMQ_DEFAULT_PASS=mypassword

  order-service:
    image: divyang25/order-service:latest
    ports:
      - "3000:3000"
    environment:
      - RABBITMQ_CONNECTION_STRING=amqp://myuser:mypassword@rabbitmq:5672/
      - PORT=3000
    depends_on:
      - rabbitmq

  product-service:
    image: divyang25/product-service:latest
    ports:
      - "3030:3030"
    environment:
      - PORT=3030

  store-front:
    image: divyang25/store-front:latest
    ports:
      - "80:80"
    depends_on:
      - product-service
      - order-service
```

### To run the application locally:

```bash
docker compose up -d
```

Then open your browser at `http://localhost`

---

## Dockerfiles

### order-service (Node.js)
```dockerfile
FROM node:20-alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
ENV PORT=3000
CMD ["node", "index.js"]
```

### product-service (Python)
```dockerfile
FROM python:3.10
WORKDIR /usr/src/app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 3030
ENV PORT=3030
CMD ["python", "app.py"]
```

### store-front (Multi-Stage Build)
```dockerfile
FROM node:20 AS build-stage
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
ENV VUE_APP_ORDER_SERVICE_URL=http://20.220.25.98:3000
ENV VUE_APP_PRODUCT_SERVICE_URL=http://20.220.25.98:3030
RUN npm run build

FROM nginx:alpine
COPY --from=build-stage /app/dist /usr/share/nginx/html
EXPOSE 80
```

---

## Key Concepts Learned

- **Docker Images & Containers** — packaging apps with all dependencies for consistent deployment
- **Multi-Stage Builds** — separating build environment from runtime to produce smaller, cleaner images
- **Docker Compose** — orchestrating multiple containers as a single application stack
- **Microservices Architecture** — each service is independently deployable and scalable
- **RabbitMQ Message Queue** — decoupling order-service from direct dependencies using async messaging
- **Azure NSG Rules** — configuring firewall inbound rules to expose container ports publicly

---


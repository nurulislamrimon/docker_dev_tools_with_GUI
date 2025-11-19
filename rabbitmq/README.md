# RabbitMQ Docker Setup Guide

This guide explains how to run **RabbitMQ (with Management UI)** using Docker and Docker Compose for a simple, beginner-friendly local development environment.

The RabbitMQ management panel provides a graphical dashboard to monitor queues, exchanges, connections, and messages.

---

## 1. Prerequisites

Ensure you have:

- **Docker**
- **Docker Compose**

Check installation:

```bash
docker --version
docker compose version
```

---

## 2. Create Project Directory

```bash
mkdir -p ~/dev-tools/rabbitmq
cd ~/dev-tools/rabbitmq
```

---

## 3. Create `docker-compose.yml`

Create a file named `docker-compose.yml` and paste the following:

```yaml
services:
  rabbitmq:
    image: rabbitmq:3-management
    container_name: dev_rabbitmq
    restart: always
    ports:
      - "5672:5672"
      - "8084:15672"
    environment:
      RABBITMQ_DEFAULT_USER: root
      RABBITMQ_DEFAULT_PASS: admin
```

---

## 4. Start RabbitMQ

Run:

```bash
docker compose up -d
```

Check running containers:

```bash
docker ps
```

You should see `dev_rabbitmq`.

---

## 5. Access RabbitMQ Management UI

Open your browser:

```
http://localhost:8084
```

### Login credentials:

- **Username:** `root`
- **Password:** `admin`

Once logged in, you can:

- Create queues
- Publish/consume messages
- Inspect channels
- Manage exchanges and bindings
- Monitor real-time throughput

---

## 6. Connecting to RabbitMQ

### AMQP URL:

```
amqp://root:admin@localhost:5672
```

### Example: Node.js (amqplib)

```bash
npm install amqplib
```

```js
import amqp from "amqplib";

async function connect() {
  const connection = await amqp.connect("amqp://root:admin@localhost:5672");
  const channel = await connection.createChannel();
  await channel.assertQueue("test");
  channel.sendToQueue("test", Buffer.from("Hello RabbitMQ!"));
}

connect();
```

---

## 7. Managing the RabbitMQ Container

### Stop containers:

```bash
docker compose down
```

### Start again:

```bash
docker compose up -d
```

### View logs:

```bash
docker logs dev_rabbitmq
```

### Follow logs live:

```bash
docker logs -f dev_rabbitmq
```

---

## 8. Troubleshooting

### Port already in use

Modify ports:

```yaml
ports:
  - "5673:5672"
  - "8090:15672"
```

New access addresses:

- RabbitMQ AMQP: `localhost:5673`
- Management UI: `http://localhost:8090`

---

### Wrong login credentials

Reset environment variables:

```yaml
RABBITMQ_DEFAULT_USER: root
RABBITMQ_DEFAULT_PASS: admin
```

Then recreate the container:

```bash
docker compose down
docker compose up -d
```

---

### Stuck queues or corrupted state

Remove container but keep settings:

```bash
docker compose down
```

Clear RabbitMQ data (if volumes were added later):

```bash
docker volume ls
docker volume rm <volume_name>
```

---

## 9. Remove Everything

Stop and remove the container:

```bash
docker compose down
```

Optionally remove image:

```bash
docker rmi rabbitmq:3-management
```

Remove directory:

```bash
rm -rf ~/dev-tools/rabbitmq
```

---

## 10. Quick Commands Reference

| Action        | Command                            |
| ------------- | ---------------------------------- |
| Start         | `docker compose up -d`             |
| Stop          | `docker compose down`              |
| Logs          | `docker logs dev_rabbitmq`         |
| RabbitMQ AMQP | `amqp://root:admin@localhost:5672` |
| Management UI | `http://localhost:8084`            |

---

Your RabbitMQ Docker environment is ready!

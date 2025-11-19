# Redis + Redis Commander Docker Setup Guide

This guide explains how to run **Redis 7** and **Redis Commander** using Docker and Docker Compose for a simple, beginner-friendly local development environment.

Redis Commander provides a web UI to inspect keys, values, TTLs, and Redis data structures.

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
mkdir -p ~/dev-tools/redis
cd ~/dev-tools/redis
```

---

## 3. Create `docker-compose.yml`

Create the file and paste:

```yaml
services:
  redis:
    image: redis:7
    container_name: dev_redis
    restart: always
    ports:
      - "6379:6379"
    command: ["redis-server", "--appendonly", "yes"]
    volumes:
      - redis_data:/data

  redis-commander:
    image: ghcr.io/joeferner/redis-commander:latest
    container_name: dev_redis_commander
    restart: always
    ports:
      - "8083:8081"
    environment:
      REDIS_HOSTS: local:redis:6379
      HTTP_USER: root
      HTTP_PASSWORD: admin
    depends_on:
      - redis

volumes:
  redis_data:
    driver: local
```

---

## 4. Start Redis + Redis Commander

Run:

```bash
docker compose up -d
```

Check containers:

```bash
docker ps
```

You should see `dev_redis` and `dev_redis_commander`.

---

## 5. Access Redis Commander (Web UI)

Open your browser:

```
http://localhost:8083
```

Credentials:

- **Username:** root
- **Password:** admin

You can:

- Explore keys
- View/edit JSON values
- Delete keys
- Check TTL
- Create new keys

---

## 6. Connect to Redis

### CLI:

```bash
redis-cli -h localhost -p 6379
```

### Test connection:

```bash
ping
```

Expected result:

```
PONG
```

---

## 7. Node.js Example

```bash
npm install redis
```

```js
import { createClient } from "redis";

const client = createClient({
  url: "redis://localhost:6379",
});

await client.connect();

await client.set("hello", "world");
console.log(await client.get("hello"));
```

---

## 8. Managing the Redis Container

Stop containers:

```bash
docker compose down
```

Start again:

```bash
docker compose up -d
```

View logs:

```bash
docker logs dev_redis
docker logs dev_redis_commander
```

Follow live logs:

```bash
docker logs -f dev_redis
```

---

## 9. Troubleshooting

### Port already in use

Change ports:

```yaml
ports:
  - "6380:6379"
  - "8095:8081"
```

Now:

- Redis: `localhost:6380`
- Redis Commander: `http://localhost:8095`

---

### Redis Commander not connecting

Make sure `REDIS_HOSTS` is correct:

```yaml
REDIS_HOSTS: local:redis:6379
```

### Data not persisting

Ensure the volume exists:

```bash
docker volume ls
docker volume inspect redis_data
```

If removed:

```bash
docker volume rm redis_data
```

→ All Redis data is lost.

---

## 10. Remove Everything

Stop containers:

```bash
docker compose down
```

Remove stored data:

```bash
docker volume rm redis_data
```

Remove images:

```bash
docker rmi redis:7 ghcr.io/joeferner/redis-commander:latest
```

Delete project:

```bash
rm -rf ~/dev-tools/redis
```

---

## 11. Quick Commands Reference

| Action                 | Command                           |
| ---------------------- | --------------------------------- |
| Start                  | `docker compose up -d`            |
| Stop                   | `docker compose down`             |
| Logs (Redis)           | `docker logs dev_redis`           |
| Logs (Redis Commander) | `docker logs dev_redis_commander` |
| Redis Host             | `localhost:6379`                  |
| Redis Commander UI     | `http://localhost:8083`           |

---

Your Redis Docker environment is ready!

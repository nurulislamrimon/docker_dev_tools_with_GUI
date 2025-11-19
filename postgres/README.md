# PostgreSQL + pgAdmin Docker Setup Guide

This guide explains how to run **PostgreSQL 16** and **pgAdmin 4** using Docker and Docker Compose for a simple, beginner-friendly local development environment.

pgAdmin provides a web UI for managing PostgreSQL databases.

---

## 1. Prerequisites

Make sure you have:

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
mkdir -p ~/dev-tools/postgres
cd ~/dev-tools/postgres
```

---

## 3. Create `docker-compose.yml`

Create a file named `docker-compose.yml` and paste the following:

```yaml
services:
  postgres:
    image: postgres:16
    container_name: dev_postgres
    restart: always
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: admin
      POSTGRES_DB: appdb
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: dev_pgadmin
    restart: always
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - "8081:80"
    volumes:
      - pgadmin_data:/var/lib/pgadmin
    depends_on:
      - postgres

volumes:
  postgres_data:
    driver: local
  pgadmin_data:
    driver: local
```

---

## 4. Start PostgreSQL + pgAdmin

Run:

```bash
docker compose up -d
```

Check containers:

```bash
docker ps
```

You should see `dev_postgres` and `dev_pgadmin`.

---

## 5. Access pgAdmin

Open your browser:

```
http://localhost:8081
```

Login with:

- **Email:** `admin@example.com`
- **Password:** `admin`

---

## 6. Connect pgAdmin to PostgreSQL

After logging into pgAdmin:

### Step 1 — Add New Server

- Right-click **Servers**
- Click **Create → Server**

### Step 2 — General Tab

Name:

```
Local Postgres
```

### Step 3 — Connection Tab

Fill in these:

| Field    | Value    |
| -------- | -------- |
| Hostname | postgres |
| Port     | 5432     |
| Username | root     |
| Password | admin    |
| Database | appdb    |

Click **Save**.

---

## 7. Connect to PostgreSQL with CLI or apps

### CLI:

```bash
psql -h localhost -U root -d appdb
```

Password: `admin`

### Node.js (pg):

```js
import { Client } from "pg";

const client = new Client({
  host: "localhost",
  user: "root",
  password: "admin",
  database: "appdb",
  port: 5432,
});

client.connect();
```

---

## 8. Managing the PostgreSQL Container

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
docker logs dev_postgres
docker logs dev_pgadmin
```

Follow logs:

```bash
docker logs -f dev_postgres
```

---

## 9. Troubleshooting

### Port already in use

Modify ports:

```yaml
ports:
  - "5433:5432"
```

Now PostgreSQL runs at:

```
localhost:5433
```

### pgAdmin not loading

Restart it:

```bash
docker restart dev_pgadmin
```

### pgAdmin cannot connect to the server

Ensure hostname is EXACTLY:

```
postgres
```

(Not `localhost` inside pgAdmin)

---

## 10. Remove Everything

Stop containers:

```bash
docker compose down
```

Remove stored data:

```bash
docker volume rm postgres_data pgadmin_data
```

Remove images:

```bash
docker rmi postgres:16 dpage/pgadmin4:latest
```

Delete project directory:

```bash
rm -rf ~/dev-tools/postgres
```

---

## 11. Quick Commands Reference

| Action          | Command                    |
| --------------- | -------------------------- |
| Start           | `docker compose up -d`     |
| Stop            | `docker compose down`      |
| Logs (Postgres) | `docker logs dev_postgres` |
| Logs (pgAdmin)  | `docker logs dev_pgadmin`  |
| PostgreSQL Host | `localhost:5432`           |
| pgAdmin         | `http://localhost:8081`    |

---

Your PostgreSQL Docker environment is ready!

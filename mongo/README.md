# MongoDB + Mongo Express Docker Setup Guide

This guide explains how to run **MongoDB** and **Mongo Express** using Docker and Docker Compose for a simple, beginner-friendly local development environment.

Mongo Express provides a web UI for viewing and managing data inside MongoDB.

---

## 1. Prerequisites

Make sure you have the following installed:

- **Docker**
- **Docker Compose**

Check installation:

```bash
docker --version
docker compose version
```

---

## 2. Create Project Directory

Create a folder to hold your MongoDB setup:

```bash
mkdir -p ~/dev-tools/mongo
cd ~/dev-tools/mongo
```

---

## 3. Create `docker-compose.yml`

Create a file named `docker-compose.yml` with the following content.

This version includes **default (no auth)** setup and shows how to **enable auth** later.

```yaml
services:
  mongo:
    image: mongo:7
    container_name: dev_mongo
    restart: always
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    # Enable authentication:
    # environment:
    #   MONGO_INITDB_ROOT_USERNAME: admin
    #   MONGO_INITDB_ROOT_PASSWORD: pass

  mongo-express:
    image: mongo-express:latest
    container_name: dev_mongo_express
    restart: always
    ports:
      - "8082:8081"
    environment:
      ME_CONFIG_MONGODB_SERVER: mongo
      # If using authentication, uncomment:
      # ME_CONFIG_MONGODB_ADMINUSERNAME: admin
      # ME_CONFIG_MONGODB_ADMINPASSWORD: pass
    depends_on:
      - mongo

volumes:
  mongo_data:
    driver: local
```

---

## 4. Start MongoDB + Mongo Express

From the same directory:

```bash
docker compose up -d
```

Check if containers are running:

```bash
docker ps
```

---

## 5. Access Mongo Express UI

Open your browser:

```
http://localhost:8082
```

Mongo Express allows you to:

- View collections
- Add/edit/delete documents
- Create databases

---

## 6. Connect to MongoDB

MongoDB runs at:

```
mongodb://localhost:27017
```

You can connect using:

- MongoDB Compass
- VSCode MongoDB extension
- Mongoose (Node.js)
- Any MongoDB client

Example Mongoose config (no auth):

```js
mongoose.connect("mongodb://localhost:27017/mydb");
```

---

# 7. Enable Authentication (Optional)

If you want secured access, follow these steps.

---

## 7.1. Enable Auth in `docker-compose.yml`

### Step 1 — Add admin credentials under `mongo`

```yaml
environment:
  MONGO_INITDB_ROOT_USERNAME: admin
  MONGO_INITDB_ROOT_PASSWORD: pass
```

### Step 2 — Add same credentials for Mongo Express

```yaml
ME_CONFIG_MONGODB_ADMINUSERNAME: admin
ME_CONFIG_MONGODB_ADMINPASSWORD: pass
```

### Step 3 — Recreate containers

```bash
docker compose down
docker compose up -d
```

---

## 7.2. Login to Mongo Express with Auth Enabled

Visit:

```
http://localhost:8082
```

Credentials:

- **Username:** `admin`
- **Password:** `pass`

---

## 7.3. Connect to MongoDB with Auth

Mongoose example:

```js
mongoose.connect("mongodb://admin:pass@localhost:27017/mydb?authSource=admin");
```

Mongo shell:

```bash
mongosh "mongodb://admin:pass@localhost:27017/?authSource=admin"
```

---

# 8. Managing the MongoDB Container

## Stop containers:

```bash
docker compose down
```

## Start containers:

```bash
docker compose up -d
```

## View logs:

```bash
docker logs dev_mongo
docker logs dev_mongo_express
```

## Follow logs live:

```bash
docker logs -f dev_mongo
```

---

# 9. Troubleshooting

### 🔹 Port already in use

Modify ports in `docker-compose.yml`:

```yaml
ports:
  - "27018:27017"
  - "8090:8081"
```

Then use:

- MongoDB: `mongodb://localhost:27018`
- Mongo Express: `http://localhost:8090`

---

### 🔹 Mongo Express cannot connect

Happens when auth is enabled but you didn’t set credentials in Mongo Express.

Fix:

```yaml
ME_CONFIG_MONGODB_ADMINUSERNAME: admin
ME_CONFIG_MONGODB_ADMINPASSWORD: pass
```

Then restart:

```bash
docker compose down
docker compose up -d
```

---

### 🔹 Data not persisting

Make sure the volume exists:

```bash
docker volume ls
docker volume inspect mongo_data
```

If you remove the volume:

```bash
docker volume rm mongo_data
```

→ All MongoDB data will be erased.

---

# 10. Remove Everything

Stop & remove:

```bash
docker compose down
```

Remove volume (deletes all data):

```bash
docker volume rm mongo_data
```

Remove images:

```bash
docker rmi mongo:7 mongo-express:latest
```

Delete project folder:

```bash
rm -rf ~/dev-tools/mongo
```

---

# 11. Quick Commands Reference

| Action               | Command                                                  |
| -------------------- | -------------------------------------------------------- |
| Start                | `docker compose up -d`                                   |
| Stop                 | `docker compose down`                                    |
| Logs (Mongo)         | `docker logs dev_mongo`                                  |
| Logs (Mongo Express) | `docker logs dev_mongo_express`                          |
| Connect no-auth      | `mongodb://localhost:27017`                              |
| Connect with auth    | `mongodb://admin:pass@localhost:27017/?authSource=admin` |
| Mongo Express        | `http://localhost:8082`                                  |

---

Enjoy your MongoDB Docker environment!

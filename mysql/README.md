# MySQL + phpMyAdmin Docker Setup Guide

This guide explains how to run **MySQL 8** and **phpMyAdmin** using Docker and Docker Compose for a simple, beginner‑friendly local development environment.

phpMyAdmin provides a web-based UI to manage your MySQL server.

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
mkdir -p ~/dev-tools/mysql
cd ~/dev-tools/mysql
```

---

## 3. Create `docker-compose.yml`

Create a file named `docker-compose.yml` and paste the following:

```yaml
services:
  mysql:
    image: mysql:8
    container_name: dev_mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: admin
      # MYSQL_DATABASE: my_app
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin/phpmyadmin:latest
    container_name: dev_phpmyadmin
    restart: always
    ports:
      - "8081:80"
    environment:
      PMA_HOST: mysql
      MYSQL_ROOT_PASSWORD: admin
      PMA_ARBITRARY: 1
    depends_on:
      - mysql

volumes:
  mysql_data:
    driver: local
```

---

## 4. Start MySQL + phpMyAdmin

Run:

```bash
docker compose up -d
```

Check containers:

```bash
docker ps
```

You should see `dev_mysql` and `dev_phpmyadmin`.

---

## 5. Access phpMyAdmin

Open:

```
http://localhost:8081
```

Login:

- **Server:** `mysql`
- **Username:** `root`
- **Password:** `admin`

---

## 6. Connect to MySQL

Connection details:

```
Host: localhost
Port: 3306
User: root
Password: admin
```

Example with MySQL CLI:

```bash
mysql -h 127.0.0.1 -u root -p
```

Enter password: `admin`

---

## 7. Create a Default Database (Optional)

Uncomment in `docker-compose.yml`:

```yaml
MYSQL_DATABASE: my_app
```

Then recreate:

```bash
docker compose down
docker compose up -d
```

A database named **my_app** will be created automatically.

---

## 8. Managing the MySQL Container

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
docker logs dev_mysql
docker logs dev_phpmyadmin
```

Follow logs:

```bash
docker logs -f dev_mysql
```

---

## 9. Troubleshooting

### Port already in use

Change ports:

```yaml
ports:
  - "3307:3306"
```

Now MySQL runs at:

```
localhost:3307
```

### phpMyAdmin cannot connect

Make sure `PMA_HOST` matches the service name:

```yaml
PMA_HOST: mysql
```

### Data not persisting

Ensure volume exists:

```bash
docker volume ls
docker volume inspect mysql_data
```

If you remove the volume:

```bash
docker volume rm mysql_data
```

→ All databases will be deleted.

---

## 10. Remove Everything

Stop containers:

```bash
docker compose down
```

Remove stored data:

```bash
docker volume rm mysql_data
```

Remove images:

```bash
docker rmi mysql:8 phpmyadmin/phpmyadmin:latest
```

Delete directory:

```bash
rm -rf ~/dev-tools/mysql
```

---

## 11. Quick Commands Reference

| Action            | Command                      |
| ----------------- | ---------------------------- |
| Start             | `docker compose up -d`       |
| Stop              | `docker compose down`        |
| Logs (MySQL)      | `docker logs dev_mysql`      |
| Logs (phpMyAdmin) | `docker logs dev_phpmyadmin` |
| MySQL Host        | `127.0.0.1:3306`             |
| phpMyAdmin        | `http://localhost:8081`      |

---

Your MySQL Docker environment is ready!

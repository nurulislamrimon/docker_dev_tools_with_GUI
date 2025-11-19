# MailHog Docker Setup Guide

This guide explains how to run [MailHog](https://github.com/mailhog/MailHog) using Docker and Docker Compose for local email testing in development environments.

---

## 1. Prerequisites

Before you start, make sure you have the following installed:

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

You can verify they are installed by running:

```bash
docker --version
docker compose version  # or: docker-compose --version
```

---

## 2. Create Project Directory

Choose or create a directory for your development tools (for example, in your home folder):

```bash
mkdir -p ~/dev-tools/mailhog
cd ~/dev-tools/mailhog
```

You can use any folder you like; just stay consistent with it.

---

## 3. Create a `docker-compose.yml` File

Inside your `mailhog` directory, create a file named `docker-compose.yml` with the following content:

```yaml
services:
  mailhog:
    image: mailhog/mailhog
    container_name: dev_mailhog
    restart: always
    ports:
      - "1025:1025" # SMTP port
      - "8085:8025" # Web UI (MailHog HTTP interface)
```

> **Note**
>
> - Port `1025` is the SMTP port (where your app will send emails).
> - Port `8085` (on your machine) maps to `8025` inside the container (MailHog web interface).
> - If port `8085` is already in use, you can change the left side of `8085:8025` to something else, e.g. `8090:8025`.

---

## 4. Start MailHog

From the directory containing `docker-compose.yml`, run:

```bash
docker compose up -d
```

or if your system uses the old `docker-compose` binary:

```bash
docker-compose up -d
```

This will:

- Pull the `mailhog/mailhog` image if not already downloaded.
- Start a container named `dev_mailhog` in the background (detached mode).

You can verify it is running with:

```bash
docker ps
```

Look for a container with the name `dev_mailhog`.

---

## 5. Access the MailHog Web UI

Once the container is running, open your browser and go to:

- **http://localhost:8085**

You should see the MailHog web interface. Any emails sent to MailHog will appear in this UI.

If you changed the mapped port, use that port instead of `8085` (e.g. `http://localhost:8090`).

---

## 6. Configure Your Application to Use MailHog

To capture emails in MailHog instead of sending real emails, configure your app's SMTP settings to use MailHog.

Typical development SMTP configuration:

- **SMTP host**: `localhost`
- **SMTP port**: `1025`
- **Username**: (leave empty)
- **Password**: (leave empty)
- **Encryption**: None (no TLS/SSL)

### Example: PHP (Laravel)

In your `.env` file:

```env
MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

### Example: Node.js (Nodemailer)

```js
const nodemailer = require("nodemailer");

const transporter = nodemailer.createTransport({
  host: "localhost",
  port: 1025,
  secure: false, // no TLS
});

transporter.sendMail({
  from: '"Test" <test@example.com>',
  to: "user@example.com",
  subject: "Hello from MailHog",
  text: "This is a test email.",
});
```

### Example: Python (Django)

In `settings.py`:

```python
EMAIL_BACKEND = "django.core.mail.backends.smtp.EmailBackend"
EMAIL_HOST = "localhost"
EMAIL_PORT = 1025
EMAIL_USE_TLS = False
EMAIL_HOST_USER = ""
EMAIL_HOST_PASSWORD = ""
```

After configuring your application, trigger an email (e.g. password reset, registration, test script) and then check the MailHog UI at `http://localhost:8085` to see it.

---

## 7. Managing the MailHog Container

### Stop MailHog

From the same directory:

```bash
docker compose down
```

or

```bash
docker-compose down
```

This stops and removes the container but keeps the image.

### Restart MailHog

If you have already created the `docker-compose.yml`, you only need to run:

```bash
docker compose up -d
```

whenever you want to start MailHog again.

### View Logs

To see the MailHog container logs:

```bash
docker logs dev_mailhog
```

To follow logs live:

```bash
docker logs -f dev_mailhog
```

---

## 8. Troubleshooting

### Port Already in Use

If you see an error about ports being in use (like `bind: address already in use`), change the ports in your `docker-compose.yml`. For example:

```yaml
ports:
  - "1026:1025"
  - "8090:8025"
```

Then update your app's configuration to match:

- SMTP host: `localhost`
- SMTP port: `1026`
- Web UI: `http://localhost:8090`

### Cannot Access Web UI

- Make sure the container is running: `docker ps`
- Check logs: `docker logs dev_mailhog`
- Ensure any local firewall is not blocking the port.

### Application Still Sending Real Emails

Double-check your application is using the MailHog SMTP host and port in the correct **environment** or **config file** (dev vs prod). Sometimes apps use different configs per environment.

---

## 9. Removing Everything

If you no longer need MailHog:

1. Stop and remove the container:

   ```bash
   cd ~/dev-tools/mailhog
   docker compose down
   ```

2. Optionally remove the image:

   ```bash
   docker rmi mailhog/mailhog
   ```

3. Optionally delete the project folder if you created one only for MailHog:

   ```bash
   cd ..
   rm -rf mailhog
   ```

---

## 10. Quick Commands Reference

- Start MailHog:

  ```bash
  docker compose up -d
  ```

- Stop MailHog:

  ```bash
  docker compose down
  ```

- See running containers:

  ```bash
  docker ps
  ```

- See logs:

  ```bash
  docker logs dev_mailhog
  ```

- Access Web UI:

  - `http://localhost:8085` (or your chosen mapped port)

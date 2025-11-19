# MinIO Docker Setup Guide

This guide explains how to run [MinIO](https://min.io/) using Docker and Docker Compose for local S3-compatible object storage in development environments.

---

## 1. Prerequisites

Before you start, make sure you have the following installed:

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

Check they are installed:

```bash
docker --version
docker compose version  # or: docker-compose --version
```

---

## 2. Create Project Directory

Create a folder to hold your MinIO setup (you can choose any path you like):

```bash
mkdir -p ~/dev-tools/minio
cd ~/dev-tools/minio
```

---

## 3. Create `docker-compose.yml`

Inside that directory, create a file named `docker-compose.yml` with the following content:

```yaml
services:
  minio:
    image: minio/minio
    container_name: dev_minio
    restart: always
    environment:
      MINIO_ROOT_USER: root
      MINIO_ROOT_PASSWORD: adminpass
    command: server /data --console-address ":9001"
    ports:
      - "9000:9000"
      - "8086:9001"
    volumes:
      - minio_data:/data

volumes:
  minio_data:
    driver: local
```

### What this configuration does

- **Image**: Uses the official `minio/minio` Docker image.
- **Container name**: `dev_minio` – makes it easy to reference in `docker` commands.
- **Restart policy**: `always` – container restarts if Docker restarts or the container exits.
- **Environment variables**:
  - `MINIO_ROOT_USER`: Root (admin) username. Here it's set to `root`.
  - `MINIO_ROOT_PASSWORD`: Root (admin) password. Here it's set to `adminpass`.
- **Command**: `server /data --console-address ":9001"`
  - Stores data in `/data` inside the container.
  - Exposes the MinIO console (web UI) on port `9001`.
- **Ports**:
  - `9000:9000` – S3 API endpoint.
  - `8086:9001` – Web console available at `http://localhost:8086`.
- **Volume**:
  - `minio_data:/data` – named Docker volume to persist your data.

> **Security note**: For local development this is fine, but **never** use weak credentials like `root/adminpass` in production.

---

## 4. Start MinIO

From the same directory as `docker-compose.yml`, run:

```bash
docker compose up -d
```

or, on older setups:

```bash
docker-compose up -d
```

This will:

- Pull the `minio/minio` image if needed.
- Start the `dev_minio` container in the background.

Check that it’s running:

```bash
docker ps
```

You should see a container called `dev_minio` with ports `9000` and `8086` exposed.

---

## 5. Access the MinIO Web Console

Open your browser and go to:

- **http://localhost:8086**

Log in with:

- **Username**: `root`
- **Password**: `adminpass`

(Or whatever values you configured in `MINIO_ROOT_USER` and `MINIO_ROOT_PASSWORD`.)

Once logged in, you can:

- Create **buckets**
- Upload/download files
- Manage access keys and users
- See server metrics

If you change the host port (e.g. `"9090:9001"`), then use that port instead of `8086`, e.g. `http://localhost:9090`.

---

## 6. S3 API Endpoint

The S3-compatible API endpoint runs on:

- **http://localhost:9000**

In all your S3 SDKs and tools (like AWS SDK, `aws-cli`, etc.), this is the endpoint you’ll use.

---

## 7. Configure Client Applications

Below are example configurations to connect to this local MinIO instance from different environments.

### 7.1. `aws-cli`

You can use `aws-cli` to interact with MinIO like you would with S3.

Create a new profile, for example `minio`:

```bash
aws configure --profile minio
```

Use these values:

- **AWS Access Key ID**: `root`
- **AWS Secret Access Key**: `adminpass`
- **Default region name**: `us-east-1` (or anything you want)
- **Default output format**: `json`

Then, in commands, specify the custom endpoint URL and the profile:

```bash
aws --profile minio --endpoint-url http://localhost:9000 s3 ls
```

You should see the list of buckets (initially empty).

### 7.2. Node.js (AWS SDK v3)

```bash
npm install @aws-sdk/client-s3
```

```js
import { S3Client, ListBucketsCommand } from "@aws-sdk/client-s3";

const s3 = new S3Client({
  region: "us-east-1",
  endpoint: "http://localhost:9000",
  forcePathStyle: true,
  credentials: {
    accessKeyId: "root",
    secretAccessKey: "adminpass",
  },
});

async function test() {
  const result = await s3.send(new ListBucketsCommand({}));
  console.log(result.Buckets);
}

test().catch(console.error);
```

Key points:

- `endpoint`: URL of your MinIO service.
- `forcePathStyle: true`: Required by MinIO (and some S3-compatible services) to use path-style URLs.

### 7.3. Python (boto3)

```bash
pip install boto3
```

```python
import boto3

s3 = boto3.client(
    "s3",
    endpoint_url="http://localhost:9000",
    aws_access_key_id="root",
    aws_secret_access_key="adminpass",
    region_name="us-east-1",
)

# List buckets
response = s3.list_buckets()
for bucket in response.get("Buckets", []):
    print(bucket["Name"])
```

### 7.4. Laravel (PHP) – Filesystem Disk

In `config/filesystems.php`, add:

```php
'disks' => [

    // ...

    'minio' => [
        'driver' => 's3',
        'key' => env('MINIO_KEY', 'root'),
        'secret' => env('MINIO_SECRET', 'adminpass'),
        'region' => env('MINIO_REGION', 'us-east-1'),
        'bucket' => env('MINIO_BUCKET', 'local-bucket'),
        'endpoint' => env('MINIO_ENDPOINT', 'http://localhost:9000'),
        'use_path_style_endpoint' => true,
    ],

],
```

In your `.env`:

```env
FILESYSTEM_DISK=minio

MINIO_KEY=root
MINIO_SECRET=adminpass
MINIO_REGION=us-east-1
MINIO_BUCKET=local-bucket
MINIO_ENDPOINT=http://localhost:9000
```

Create the bucket `local-bucket` using the MinIO console first, then you can use Laravel's Storage facade:

```php
use Illuminate\Support\Facades\Storage;

Storage::put('example.txt', 'Hello MinIO!');
```

---

## 8. Managing the MinIO Container

### Stop MinIO

```bash
docker compose down
```

or

```bash
docker-compose down
```

This stops and removes the container but keeps the volume `minio_data` and the image.

### Start MinIO Again

```bash
docker compose up -d
```

MinIO will start again and reuse the same `minio_data` volume (your buckets and objects remain).

### View Logs

To see logs:

```bash
docker logs dev_minio
```

To follow logs in real time:

```bash
docker logs -f dev_minio
```

---

## 9. Troubleshooting

### Port Already in Use

If port `9000` or `8086` is already used by another service, change the host port mapping in `docker-compose.yml`.

Example:

```yaml
ports:
  - "9002:9000"
  - "9090:9001"
```

Then:

- S3 endpoint: `http://localhost:9002`
- Web console: `http://localhost:9090`

Update your SDKs and tools to use the new endpoint.

### Cannot Log In

- Make sure you are using the exact values set in `MINIO_ROOT_USER` and `MINIO_ROOT_PASSWORD`.
- Recreate the container if you change these environment variables:

  ```bash
  docker compose down
  docker compose up -d
  ```

### Data Not Persisting

- Ensure the `minio_data` volume is defined and mounted correctly.
- Check with:

  ```bash
  docker volume ls
  docker volume inspect minio_data
  ```

If you remove the volume (e.g. `docker volume rm minio_data`), all stored objects will be deleted.

---

## 10. Removing Everything

If you no longer need this MinIO setup:

1. Stop and remove containers:

   ```bash
   cd ~/dev-tools/minio
   docker compose down
   ```

2. Remove the data volume (this permanently deletes all objects):

   ```bash
   docker volume rm minio_data
   ```

3. Optionally remove the image:

   ```bash
   docker rmi minio/minio
   ```

4. Optionally delete the project directory:

   ```bash
   cd ..
   rm -rf minio
   ```

---

## 11. Quick Commands Reference

- Start MinIO:

  ```bash
  docker compose up -d
  ```

- Stop MinIO:

  ```bash
  docker compose down
  ```

- Check running containers:

  ```bash
  docker ps
  ```

- View logs:

  ```bash
  docker logs dev_minio
  ```

- Web console:

  ```text
  http://localhost:8086
  ```

- S3 API endpoint:

  ```text
  http://localhost:9000
  ```

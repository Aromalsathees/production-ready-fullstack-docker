# 🐳 Full-Stack Dockerization

### Django REST Framework + React + PostgreSQL + Docker Compose

This repository demonstrates how to **Dockerize a complete full-stack application** using:

* 🐍 Django REST Framework
* ⚛️ React
* 🐘 PostgreSQL
* 🐳 Docker
* 🐳 Docker Compose
* 🌐 Nginx

The goal of this project is to understand how a full-stack application works inside Docker using separate containers for the **backend, frontend, and database**.

---

# 🏗️ Architecture

```text
                Full-Stack Application
                        │
              Docker Compose
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
   PostgreSQL        Django          React
      DB              API           Frontend
        │               │               │
        │               │               ▼
        │               │            Nginx
        │               │
        └───────────────┴───────────────
                 Docker Network
```

---

# 📁 Project Structure

```text
fullstack-docker-deployment/
│
├── backend-drf/
│   ├── Dockerfile
│   ├── .env.docker
│   ├── .env.production
│   ├── requirements.txt
│   ├── manage.py
│   └── ...
│
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── ...
│
├── docker-compose.yml
├── .gitignore
└── README.md
```

---

# 🧰 Prerequisites

Install the following:

* Git
* Python 3.10+
* Node.js & npm
* Docker Desktop
* VS Code (recommended)

Verify Docker:

```bash
docker --version
docker compose version
```

---

# 💻 Step 1 — Run the Project Locally

Before Dockerizing the project, make sure the application works normally without Docker.

## Backend

Go to the backend:

```bash
cd backend-drf
```

Create a virtual environment.

### Windows

```bash
python -m venv env
env\Scripts\activate
```

### Mac / Linux

```bash
python3 -m venv env
source env/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Create `.env`:

```env
DEBUG=True
SECRET_KEY=<YOUR-SECRET-KEY>

DB_NAME=<DATABASE-NAME>
DB_USER=<POSTGRES-USERNAME>
DB_PASSWORD=<YOUR-PASSWORD>
DB_HOST=localhost
DB_PORT=5432
```

Run migrations:

```bash
python manage.py migrate
```

Start Django:

```bash
python manage.py runserver
```

Backend:

```text
http://127.0.0.1:8000/
```

---

# ⚛️ Step 2 — Run React Locally

Open another terminal:

```bash
cd frontend
```

Install packages:

```bash
npm install
```

Create:

```text
frontend/.env
```

Add:

```env
VITE_SERVER_BASE_URL=http://127.0.0.1:8000/api/v1
```

Start React:

```bash
npm run dev
```

Frontend:

```text
http://localhost:5173/
```

Make sure the complete application works before moving to Docker.

---

# 🐳 Step 3 — Create Backend Dockerfile

Create:

```text
backend-drf/Dockerfile
```

```dockerfile
FROM python:3.10-slim

ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["gunicorn", "clickmart_main.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "3", "--timeout", "180"]
```

### What happens here?

```text
Python Image
     ↓
Create /app
     ↓
Install system dependencies
     ↓
Copy requirements.txt
     ↓
Install Python packages
     ↓
Copy Django project
     ↓
Run Django application
```

---

# ⚛️ Step 4 — Create Frontend Dockerfile

Create:

```text
frontend/Dockerfile
```

```dockerfile
# Stage 1: Build React
FROM node:18 AS build

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

ARG VITE_SERVER_BASE_URL

ENV VITE_SERVER_BASE_URL=$VITE_SERVER_BASE_URL

RUN npm run build


# Stage 2: Nginx
FROM nginx:alpine

COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### What happens?

```text
Node Container
      ↓
npm install
      ↓
React Build
      ↓
dist/
      ↓
Nginx Container
      ↓
Serve React Application
```

---

# 🐘 Step 5 — PostgreSQL Configuration

Create:

```text
backend-drf/.env.production
```

```env
POSTGRES_DB=<YOUR-DATABASE-NAME>
POSTGRES_USER=postgres
POSTGRES_PASSWORD=<YOUR-PASSWORD>
```

These credentials are used by the PostgreSQL container.

---

# 🔐 Step 6 — Docker Backend Environment

Create:

```text
backend-drf/.env.docker
```

```env
SECRET_KEY=<YOUR-DJANGO-SECRETKEY>
DEBUG=True

DB_NAME=<YOUR-DATABASE-NAME>
DB_USER=postgres
DB_PASSWORD=<YOUR-PASSWORD>
DB_HOST=db
DB_PORT=5432
```

### Important

For local development:

```env
DB_HOST=localhost
```

For Docker:

```env
DB_HOST=db
```

`db` is the PostgreSQL **service name** in Docker Compose.

---

# 🐳 Step 7 — Create Docker Compose

Create `docker-compose.yml` in the root directory:

```yaml
services:

  db:
    image: postgres:16-alpine
    env_file:
      - ./backend-drf/.env.production
    volumes:
      - postgres_data:/var/lib/postgresql/data


  backend:
    build: ./backend-drf

    ports:
      - "8000:8000"

    env_file:
      - ./backend-drf/.env.docker

    depends_on:
      - db

    volumes:
      - ./backend-drf/static:/app/static
      - ./backend-drf/media:/app/media

    command: >
      sh -c "python manage.py collectstatic --noinput &&
             python manage.py migrate &&
             python manage.py runserver 0.0.0.0:8000"


  frontend:
    build:
      context: ./frontend
      args:
        VITE_SERVER_BASE_URL: "http://backend:8000/api/v1"

    ports:
      - "5173:80"

    depends_on:
      - backend


volumes:
  postgres_data:
```

---

# 🚀 Step 8 — Dockerize the Application

From the root directory:

```bash
docker compose up --build
```

Docker will:

```text
        docker compose
              │
     ┌────────┼────────┐
     ▼        ▼        ▼
 PostgreSQL Django    React
   Container Container Container
              │
              ▼
            Nginx
```

---

# 🔍 Step 9 — Check Containers

Run:

```bash
docker compose ps
```

You should see:

```text
db
backend
frontend
```

---

# 🌐 Step 10 — Access the Application

Backend:

```text
http://localhost:8000
```

Frontend:

```text
http://localhost:5173
```

---

# 👤 Step 11 — Create Django Superuser

Create a superuser inside the backend container:

```bash
docker compose exec backend python manage.py createsuperuser
```

Follow the instructions shown in the terminal.

---

# 📋 Useful Docker Commands

### Start the application

```bash
docker compose up
```

### Build and start

```bash
docker compose up --build
```

### Start in background

```bash
docker compose up -d
```

### Stop containers

```bash
docker compose stop
```

### Start existing containers

```bash
docker compose start
```

### Stop and remove containers

```bash
docker compose down
```

### Stop and remove containers + volumes

```bash
docker compose down -v
```

> ⚠️ `-v` removes the PostgreSQL Docker volume. Database data may be lost.

### Check containers

```bash
docker compose ps
```

### View logs

```bash
docker compose logs
```

### Backend logs

```bash
docker compose logs backend
```

### Follow backend logs

```bash
docker compose logs -f backend
```

### Enter backend container

```bash
docker compose exec backend sh
```

---

# 🔄 Understanding Code Changes

During development, Docker volumes can be used to synchronize project files between the host machine and the container.

For example:

```yaml
volumes:
  - ./backend-drf:/app
```

This means:

```text
Your Computer              Container

backend-drf/  ←────────→    /app
```

Changes made to the project can then be available inside the running container without rebuilding the image for every code change.

However, files such as `requirements.txt`, `Dockerfile`, or other build-related configuration changes may require:

```bash
docker compose up --build
```

---

# 💾 Understanding PostgreSQL Volume

The Compose file contains:

```yaml
volumes:
  postgres_data:
```

This creates a persistent Docker volume for PostgreSQL.

```text
PostgreSQL Container
        │
        ▼
postgres_data
        │
        ▼
Database data persists
```

So removing/recreating the PostgreSQL container does not automatically remove the database volume.

To completely reset the database:

```bash
docker compose down -v
```

---

# 🎯 What You Will Learn

By completing this project, you will understand:

* Full-stack application structure
* Django REST Framework
* React
* PostgreSQL
* Docker images
* Docker containers
* Dockerfiles
* Docker Compose
* Environment variables
* Docker networking
* Docker volumes
* Multi-stage Docker builds
* Nginx for serving React
* Running a complete full-stack application using Docker

---

# 🚀 Final Goal

The goal is to understand how to take a normal full-stack application:

```text
Django + React + PostgreSQL
```

and convert it into:

```text

                 Dockerfile
                     ↓
                  build
                     ↓
                   Image
                     ↓
             docker compose up
                     ↓
                Container




                 docker-compose.yml
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
      Backend         Frontend          DB
          ↓              ↓              ↓
    Dockerfile      Dockerfile      postgres image
          ↓              ↓              ↓
    Backend Image   Frontend Image   PostgreSQL Image
          ↓              ↓              ↓
    Backend          Frontend        PostgreSQL
    Container        Container       Container


```

After completing this repository, you should be able to **Dockerize your own Django + React + PostgreSQL projects**.

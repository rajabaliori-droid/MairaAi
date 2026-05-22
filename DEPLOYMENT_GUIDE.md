# MYRA Deployment Guide

## Architecture Overview

```
[Android App] ←──WebSocket──→ [Nginx] ←──→ [FastAPI Backend]
                                               ├── PostgreSQL
                                               └── Redis Cache
```

---

## 1. Docker Compose (Recommended for Self-Hosting)

```bash
# 1. Clone repo
git clone https://github.com/your-org/myra.git && cd myra

# 2. Configure environment
cp backend/.env.example backend/.env
nano backend/.env
# Set: OPENAI_API_KEY, JWT_SECRET_KEY, DATABASE_URL, REDIS_URL

# 3. Start all services
docker-compose up -d

# 4. Check status
docker-compose ps
docker-compose logs api

# 5. Test health
curl https://your-domain.com/health/
```

---

## 2. Cloud Deployment (AWS / GCP / Azure)

### AWS EC2 Quickstart
```bash
# Install Docker
sudo apt update && sudo apt install -y docker.io docker-compose-plugin
sudo systemctl start docker && sudo usermod -aG docker ubuntu

# Clone and deploy
git clone https://github.com/your-org/myra.git
cd myra
cp backend/.env.example backend/.env
# Edit .env with production values

docker compose up -d
```

### SSL / HTTPS (Let's Encrypt)
```bash
# Install certbot
sudo apt install certbot python3-certbot-nginx

# Issue certificate
sudo certbot --nginx -d api.myra.ai

# Auto-renew
sudo crontab -e
# Add: 0 0 * * * certbot renew --quiet
```

---

## 3. Environment Variables (Production)

```env
APP_ENV=production
DEBUG=false
HOST=0.0.0.0
PORT=8000
WORKERS=4

# Use strong, random values:
JWT_SECRET_KEY=$(openssl rand -hex 64)

DATABASE_URL=postgresql+asyncpg://myra:STRONG_PASSWORD@localhost:5432/myra_db
REDIS_URL=redis://:REDIS_PASSWORD@localhost:6379/0

OPENAI_API_KEY=sk-your-production-key

# Lock down CORS:
ALLOWED_ORIGINS=https://app.myra.ai
```

---

## 4. Database Migrations

```bash
# Initialize Alembic (first time)
cd backend
alembic init alembic

# Create migration
alembic revision --autogenerate -m "initial schema"

# Apply migrations
alembic upgrade head
```

---

## 5. Scaling

For high-traffic deployments:
- Use `WORKERS=4` or more in `.env`
- Put Nginx in front as reverse proxy
- Use managed PostgreSQL (AWS RDS, Cloud SQL)
- Use managed Redis (ElastiCache, Memorystore)
- Deploy with Kubernetes for auto-scaling

---

## 6. Monitoring

```bash
# View live logs
docker-compose logs -f api

# Resource usage
docker stats

# Prometheus metrics available at:
# GET /metrics  (if prometheus-fastapi-instrumentator enabled)
```

---

## 7. Backup

```bash
# Backup PostgreSQL
docker exec myra_postgres pg_dump -U myra myra_db > backup_$(date +%Y%m%d).sql

# Restore
docker exec -i myra_postgres psql -U myra myra_db < backup_20240101.sql
```

---

## 8. App Distribution

### Option A: Direct APK (Sideload)
```bash
# Build release APK
cd android && ./gradlew assembleRelease

# Host APK at:
# https://your-domain.com/myra-release.apk

# Users enable "Unknown Sources" and install
```

### Option B: Google Play Store
```bash
# Build AAB
cd android && ./gradlew bundleRelease

# Upload android/app/build/outputs/bundle/release/app-release.aab
# to Google Play Console
```

---

## 9. CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/build.yml
name: Build MYRA

on:
  push:
    branches: [main]

jobs:
  build-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with: { python-version: '3.11' }
      - run: pip install -r backend/requirements.txt
      - run: pytest backend/tests/

  build-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { distribution: 'temurin', java-version: '17' }
      - uses: actions/setup-node@v4
        with: { node-version: '18' }
      - run: npm install
      - run: cd android && ./gradlew assembleRelease
      - uses: actions/upload-artifact@v4
        with:
          name: myra-apk
          path: android/app/build/outputs/apk/release/app-release.apk
```

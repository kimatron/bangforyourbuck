# Deployment Guide - Bang For Your Buck

This guide covers deploying Bang For Your Buck to production using various hosting platforms.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
- [Backend Deployment](#backend-deployment)
  - [Railway](#railway-recommended)
  - [Render](#render)
  - [DigitalOcean](#digitalocean)
- [Frontend Deployment](#frontend-deployment)
  - [Vercel](#vercel-recommended)
  - [Netlify](#netlify)
- [Database Setup](#database-setup)
- [Redis Setup](#redis-setup)
- [Media Storage (Cloudinary)](#media-storage-cloudinary)
- [Domain Configuration](#domain-configuration)
- [SSL Certificates](#ssl-certificates)
- [Monitoring & Logging](#monitoring--logging)
- [CI/CD Pipeline](#cicd-pipeline)
- [Backup Strategy](#backup-strategy)
- [Scaling Considerations](#scaling-considerations)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before deploying, ensure you have:

- [ ] GitHub repository with your code
- [ ] Domain name (bangforyourbuck.ie)
- [ ] Google Maps API key
- [ ] Cloudinary account
- [ ] Sentry account (optional but recommended)
- [ ] Email service account (SendGrid/Mailgun)

---

## Environment Setup

### Required Environment Variables

#### Backend (.env)
```bash
# Django Core
SECRET_KEY=your-production-secret-key-here
DEBUG=False
ALLOWED_HOSTS=api.bangforyourbuck.ie,bangforyourbuck.ie

# Database
DATABASE_URL=postgresql://user:password@host:5432/bangforyourbuck

# Redis
REDIS_URL=redis://default:password@host:6379

# Security
SECURE_SSL_REDIRECT=True
SESSION_COOKIE_SECURE=True
CSRF_COOKIE_SECURE=True
SECURE_HSTS_SECONDS=31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS=True
SECURE_HSTS_PRELOAD=True

# CORS
CORS_ALLOWED_ORIGINS=https://bangforyourbuck.ie,https://www.bangforyourbuck.ie

# Cloudinary
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

# Email
EMAIL_BACKEND=django.core.mail.backends.smtp.EmailBackend
EMAIL_HOST=smtp.sendgrid.net
EMAIL_PORT=587
EMAIL_USE_TLS=True
EMAIL_HOST_USER=apikey
EMAIL_HOST_PASSWORD=your_sendgrid_api_key
DEFAULT_FROM_EMAIL=noreply@bangforyourbuck.ie

# Sentry
SENTRY_DSN=https://your-sentry-dsn@sentry.io/project-id
SENTRY_ENVIRONMENT=production

# Celery
CELERY_BROKER_URL=redis://default:password@host:6379/0
CELERY_RESULT_BACKEND=redis://default:password@host:6379/0
```

#### Frontend (.env.production)
```bash
# API
VITE_API_BASE_URL=https://api.bangforyourbuck.ie/api/v1

# Google Maps
VITE_GOOGLE_MAPS_API_KEY=your_google_maps_key

# Sentry
VITE_SENTRY_DSN=https://your-frontend-sentry-dsn@sentry.io/project-id
VITE_SENTRY_ENVIRONMENT=production

# Analytics
VITE_GA_TRACKING_ID=G-XXXXXXXXXX
```

---

## Backend Deployment

### Railway (Recommended)

Railway offers the simplest deployment with built-in PostgreSQL and Redis.

#### Step 1: Install Railway CLI
```bash
# macOS
brew install railway

# Or via npm
npm install -g @railway/cli
```

#### Step 2: Login and Initialize
```bash
# Login to Railway
railway login

# Initialize project
cd backend
railway init
```

#### Step 3: Add Services
```bash
# Create PostgreSQL database
railway add --plugin postgresql

# Create Redis instance
railway add --plugin redis

# Link services to your project
railway link
```

#### Step 4: Configure Environment Variables
```bash
# Set environment variables via CLI
railway variables set SECRET_KEY="your-secret-key"
railway variables set DEBUG="False"
railway variables set ALLOWED_HOSTS="api.bangforyourbuck.ie"

# Or use the Railway dashboard
railway open
# Navigate to Variables tab and add all variables
```

#### Step 5: Create Procfile
Create `Procfile` in backend directory:
```
web: gunicorn config.wsgi:application --bind 0.0.0.0:$PORT
worker: celery -A config worker --loglevel=info
beat: celery -A config beat --loglevel=info
```

#### Step 6: Create railway.json
Create `railway.json` in backend directory:
```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "NIXPACKS",
    "buildCommand": "pip install -r requirements.txt && python manage.py collectstatic --noinput && python manage.py migrate"
  },
  "deploy": {
    "startCommand": "gunicorn config.wsgi:application",
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

#### Step 7: Deploy
```bash
# Deploy to Railway
railway up

# Check deployment status
railway status

# View logs
railway logs
```

#### Step 8: Enable Custom Domain
```bash
# Add custom domain
railway domain

# Or via dashboard:
# Settings > Networking > Custom Domain
# Add: api.bangforyourbuck.ie
```

#### Step 9: Run Migrations and Create Superuser
```bash
# SSH into Railway instance
railway run python manage.py migrate
railway run python manage.py createsuperuser
railway run python manage.py loaddata fixtures/seed_data.json
```

---

### Render

Alternative to Railway with similar features.

#### Step 1: Create render.yaml
Create `render.yaml` in project root:
```yaml
services:
  - type: web
    name: bangforbuck-api
    env: python
    region: frankfurt
    plan: starter
    buildCommand: "pip install -r requirements.txt && python manage.py collectstatic --noinput && python manage.py migrate"
    startCommand: "gunicorn config.wsgi:application"
    envVars:
      - key: PYTHON_VERSION
        value: 3.11.0
      - key: DATABASE_URL
        fromDatabase:
          name: bangforbuck-db
          property: connectionString
      - key: REDIS_URL
        fromService:
          type: redis
          name: bangforbuck-redis
          property: connectionString
      - key: SECRET_KEY
        generateValue: true
      - key: DEBUG
        value: False

  - type: worker
    name: bangforbuck-celery
    env: python
    buildCommand: "pip install -r requirements.txt"
    startCommand: "celery -A config worker --loglevel=info"
    envVars:
      - key: DATABASE_URL
        fromDatabase:
          name: bangforbuck-db
          property: connectionString
      - key: REDIS_URL
        fromService:
          type: redis
          name: bangforbuck-redis
          property: connectionString

databases:
  - name: bangforbuck-db
    databaseName: bangforyourbuck
    user: bangforbuck_user
    plan: starter
    region: frankfurt
    postgresMajorVersion: 15

services:
  - type: redis
    name: bangforbuck-redis
    plan: starter
    region: frankfurt
    maxmemoryPolicy: noeviction
```

#### Step 2: Connect GitHub Repository
1. Go to [render.com/dashboard](https://render.com/dashboard)
2. Click "New +" > "Blueprint"
3. Connect your GitHub repository
4. Render will detect render.yaml and create all services

#### Step 3: Configure Environment Variables
Add remaining environment variables in Render Dashboard:
- ALLOWED_HOSTS
- CORS_ALLOWED_ORIGINS
- CLOUDINARY credentials
- Email settings
- Sentry DSN

#### Step 4: Enable Custom Domain
1. Go to your web service
2. Settings > Custom Domain
3. Add: api.bangforyourbuck.ie
4. Update DNS records as instructed

---

### DigitalOcean

For more control and cost efficiency at scale.

#### Step 1: Create Droplet
```bash
# Via doctl CLI
doctl compute droplet create bangforbuck \
  --region fra1 \
  --size s-2vcpu-4gb \
  --image ubuntu-22-04-x64 \
  --ssh-keys your-ssh-key-id
```

Or via web interface:
1. Go to DigitalOcean Dashboard
2. Create > Droplets
3. Choose Ubuntu 22.04
4. Select Frankfurt region
5. Choose 2 vCPU, 4GB RAM plan

#### Step 2: Initial Server Setup
```bash
# SSH into droplet
ssh root@your-droplet-ip

# Update system
apt update && apt upgrade -y

# Create deploy user
adduser deploy
usermod -aG sudo deploy
su - deploy

# Install dependencies
sudo apt install -y python3.11 python3-pip python3-venv nginx postgresql postgresql-contrib redis-server git

# Install Docker (for containerized deployment)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker deploy
```

#### Step 3: Set Up PostgreSQL with PostGIS
```bash
# Install PostGIS
sudo apt install -y postgis postgresql-15-postgis-3

# Configure PostgreSQL
sudo -u postgres psql

# In PostgreSQL prompt:
CREATE DATABASE bangforyourbuck;
CREATE USER bangforbuck_user WITH PASSWORD 'your-password';
ALTER ROLE bangforbuck_user SET client_encoding TO 'utf8';
ALTER ROLE bangforbuck_user SET default_transaction_isolation TO 'read committed';
ALTER ROLE bangforbuck_user SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE bangforyourbuck TO bangforbuck_user;

# Connect to database
\c bangforyourbuck

# Enable PostGIS
CREATE EXTENSION postgis;

# Exit
\q
```

#### Step 4: Deploy with Docker Compose
Create `docker-compose.prod.yml`:
```yaml
version: '3.9'

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    command: gunicorn config.wsgi:application --bind 0.0.0.0:8000 --workers 4
    volumes:
      - static_volume:/app/staticfiles
      - media_volume:/app/media
    expose:
      - 8000
    env_file:
      - ./backend/.env.prod
    depends_on:
      - db
      - redis
    restart: unless-stopped

  celery:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    command: celery -A config worker --loglevel=info
    env_file:
      - ./backend/.env.prod
    depends_on:
      - db
      - redis
    restart: unless-stopped

  celery-beat:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    command: celery -A config beat --loglevel=info
    env_file:
      - ./backend/.env.prod
    depends_on:
      - db
      - redis
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - static_volume:/app/staticfiles:ro
      - media_volume:/app/media:ro
    depends_on:
      - backend
    restart: unless-stopped

  db:
    image: postgis/postgis:15-3.3
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    environment:
      - POSTGRES_DB=bangforyourbuck
      - POSTGRES_USER=bangforbuck_user
      - POSTGRES_PASSWORD=your-password
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    restart: unless-stopped

volumes:
  postgres_data:
  static_volume:
  media_volume:
```

Create `backend/Dockerfile.prod`:
```dockerfile
FROM python:3.11-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Set work directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    postgresql-client \
    gdal-bin \
    libgdal-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt /app/
RUN pip install --upgrade pip && pip install -r requirements.txt

# Copy project
COPY . /app/

# Collect static files
RUN python manage.py collectstatic --noinput

# Run as non-root user
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000"]
```

#### Step 5: Configure Nginx
Create `nginx/nginx.conf`:
```nginx
upstream backend {
    server backend:8000;
}

server {
    listen 80;
    server_name api.bangforyourbuck.ie;

    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.bangforyourbuck.ie;

    # SSL Configuration
    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    # Security headers
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options DENY;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # Client body size
    client_max_body_size 10M;

    # Static files
    location /static/ {
        alias /app/staticfiles/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # Media files
    location /media/ {
        alias /app/media/;
        expires 7d;
    }

    # Proxy to Django
    location / {
        proxy_pass http://backend;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;
        proxy_redirect off;
    }
}
```

#### Step 6: Deploy
```bash
# Clone repository
cd /home/deploy
git clone https://github.com/yourusername/bangforyourbuck.git
cd bangforyourbuck

# Create environment file
cp backend/.env.example backend/.env.prod
# Edit .env.prod with production values

# Build and start services
docker-compose -f docker-compose.prod.yml up -d --build

# Run migrations
docker-compose -f docker-compose.prod.yml exec backend python manage.py migrate

# Create superuser
docker-compose -f docker-compose.prod.yml exec backend python manage.py createsuperuser

# Load seed data
docker-compose -f docker-compose.prod.yml exec backend python manage.py loaddata fixtures/seed_data.json
```

#### Step 7: Set Up SSL with Let's Encrypt
```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d api.bangforyourbuck.ie

# Auto-renewal is set up automatically
# Test renewal:
sudo certbot renew --dry-run
```

---

## Frontend Deployment

### Vercel (Recommended)

Vercel is optimized for React applications.

#### Step 1: Install Vercel CLI
```bash
npm install -g vercel
```

#### Step 2: Configure vercel.json
Create `frontend/vercel.json`:
```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "vite",
  "regions": ["fra1"],
  "env": {
    "VITE_API_BASE_URL": "https://api.bangforyourbuck.ie/api/v1",
    "VITE_GOOGLE_MAPS_API_KEY": "@google_maps_api_key",
    "VITE_SENTRY_DSN": "@sentry_dsn_frontend"
  },
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-XSS-Protection",
          "value": "1; mode=block"
        }
      ]
    }
  ],
  "rewrites": [
    {
      "source": "/(.*)",
      "destination": "/index.html"
    }
  ]
}
```

#### Step 3: Deploy
```bash
cd frontend

# Login to Vercel
vercel login

# Deploy
vercel --prod
```

Or connect via GitHub:
1. Go to [vercel.com](https://vercel.com)
2. Import your GitHub repository
3. Select frontend directory as root
4. Add environment variables
5. Deploy

#### Step 4: Configure Custom Domain
1. Go to Project Settings > Domains
2. Add: bangforyourbuck.ie and www.bangforyourbuck.ie
3. Update DNS records as instructed

---

### Netlify

Alternative to Vercel with similar features.

#### Step 1: Create netlify.toml
Create `frontend/netlify.toml`:
```toml
[build]
  base = "frontend"
  publish = "dist"
  command = "npm run build"

[build.environment]
  NODE_VERSION = "18"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-XSS-Protection = "1; mode=block"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"
```

#### Step 2: Deploy via Netlify CLI
```bash
# Install Netlify CLI
npm install -g netlify-cli

# Login
netlify login

# Initialize
cd frontend
netlify init

# Deploy
netlify deploy --prod
```

Or connect via web interface:
1. Go to [app.netlify.com](https://app.netlify.com)
2. New site from Git
3. Connect repository
4. Configure build settings
5. Deploy

---

## Database Setup

### PostgreSQL with PostGIS

#### Backup & Restore
```bash
# Backup
pg_dump -U bangforbuck_user -h localhost bangforyourbuck > backup.sql

# Restore
psql -U bangforbuck_user -h localhost bangforyourbuck < backup.sql
```

#### Performance Tuning
Edit `/etc/postgresql/15/main/postgresql.conf`:
```conf
# Memory
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 256MB
work_mem = 32MB

# Connections
max_connections = 100

# Logging
log_min_duration_statement = 1000  # Log slow queries

# Autovacuum
autovacuum = on
autovacuum_max_workers = 3
```

Restart PostgreSQL:
```bash
sudo systemctl restart postgresql
```

---

## Redis Setup

### Configuration

Edit `/etc/redis/redis.conf`:
```conf
# Security
requirepass your-redis-password
bind 127.0.0.1

# Performance
maxmemory 256mb
maxmemory-policy allkeys-lru

# Persistence
save 900 1
save 300 10
save 60 10000
```

Restart Redis:
```bash
sudo systemctl restart redis
```

---

## Media Storage (Cloudinary)

### Setup

1. Sign up at [cloudinary.com](https://cloudinary.com)
2. Get credentials from Dashboard
3. Add to environment variables
4. Configure upload presets in Cloudinary Dashboard

### Django Settings

Already configured in `settings.py`:
```python
CLOUDINARY_STORAGE = {
    'CLOUD_NAME': os.getenv('CLOUDINARY_CLOUD_NAME'),
    'API_KEY': os.getenv('CLOUDINARY_API_KEY'),
    'API_SECRET': os.getenv('CLOUDINARY_API_SECRET'),
}

DEFAULT_FILE_STORAGE = 'cloudinary_storage.storage.MediaCloudinaryStorage'
```

### Upload Presets

Create upload presets in Cloudinary Dashboard:

**receipt_photos:**
- Folder: `receipts/`
- Format: Auto
- Quality: Auto
- Transformation: `c_limit,w_1200,h_1600,q_auto:good`

**venue_images:**
- Folder: `venues/`
- Format: Auto
- Quality: Auto
- Transformation: `c_fill,w_800,h_600,q_auto:good`

---

## Domain Configuration

### DNS Records

Configure your DNS at your registrar (e.g., Cloudflare, GoDaddy):

#### For Frontend (bangforyourbuck.ie)
```
Type: A
Name: @
Value: <Vercel IP or CNAME>
TTL: Auto

Type: CNAME
Name: www
Value: cname.vercel-dns.com
TTL: Auto
```

#### For Backend API (api.bangforyourbuck.ie)
```
Type: A
Name: api
Value: <Railway/Render/DigitalOcean IP>
TTL: Auto Cloudflare Specific Settings
If using Cloudflare:

SSL/TLS > Overview > Set to "Full (strict)"
SSL/TLS > Edge Certificates > Always Use HTTPS: ON
Speed > Optimization > Auto Minify: Enable all
Caching > Configuration > Browser Cache TTL: 4 hours


SSL Certificates
Automated (Railway/Render/Vercel)
SSL is automatically provisioned and renewed.
Manual (Let's Encrypt on DigitalOcean)
bash# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain certificates
sudo certbot --nginx -d api.bangforyourbuck.ie

# Certificates stored at:
# /etc/letsencrypt/live/api.bangforyourbuck.ie/fullchain.pem
# /etc/letsencrypt/live/api.bangforyourbuck.ie/privkey.pem

# Auto-renewal (runs twice daily)
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer

# Test renewal
sudo certbot renew --dry-run
Certificate Monitoring
Set up monitoring to alert before expiration:
bash# Add to crontab
0 0 1 * * certbot renew --post-hook "systemctl reload nginx" && curl https://hc-ping.com/your-uuid

Monitoring & Logging
Sentry Setup
Backend (Django)
Install SDK:
bashpip install sentry-sdk
Configure in settings.py:
pythonimport sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration

sentry_sdk.init(
    dsn=os.getenv('SENTRY_DSN'),
    integrations=[DjangoIntegration()],
    environment=os.getenv('SENTRY_ENVIRONMENT', 'production'),
    traces_sample_rate=0.1,
    send_default_pii=False,
)
Frontend (React)
Install SDK:
bashnpm install @sentry/react @sentry/tracing
Configure in src/main.tsx:
typescriptimport * as Sentry from "@sentry/react";
import { BrowserTracing } from "@sentry/tracing";

Sentry.init({
  dsn: import.meta.env.VITE_SENTRY_DSN,
  environment: import.meta.env.VITE_SENTRY_ENVIRONMENT,
  integrations: [new BrowserTracing()],
  tracesSampleRate: 0.1,
});
UptimeRobot

Sign up at uptimerobot.com
Add monitors:

Frontend: https://bangforyourbuck.ie (HTTP(s), 5 min interval)
Backend API: https://api.bangforyourbuck.ie/api/health/ (HTTP(s), 5 min interval)


Set up alert contacts (email, Slack, SMS)

Application Logs
Backend Logging
Configure in settings.py:
pythonLOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': '/var/log/bangforbuck/django.log',
            'maxBytes': 1024 * 1024 * 10,  # 10MB
            'backupCount': 5,
            'formatter': 'verbose',
        },
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file', 'console'],
            'level': 'INFO',
            'propagate': True,
        },
        'apps': {
            'handlers': ['file', 'console'],
            'level': 'INFO',
            'propagate': False,
        },
    },
}
View Logs
Railway:
bashrailway logs
Render:
Via dashboard: Logs tab
DigitalOcean (Docker):
bashdocker-compose -f docker-compose.prod.yml logs -f backend
docker-compose -f docker-compose.prod.yml logs -f celery
Database Monitoring
PostgreSQL Slow Query Log
Enable in postgresql.conf:
conflog_min_duration_statement = 1000  # Log queries > 1s
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '
log_statement = 'all'  # Or 'ddl' for schema changes only
Monitor slow queries:
bash# On server
tail -f /var/log/postgresql/postgresql-15-main.log | grep "duration"
Django Debug Toolbar (Development Only)
Never enable in production! Use locally to identify slow queries.

CI/CD Pipeline
GitHub Actions
Create .github/workflows/deploy.yml:
yamlname: Deploy to Production

on:
  push:
    branches:
      - main

jobs:
  test-backend:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgis/postgis:15-3.3
        env:
          POSTGRES_DB: test_db
          POSTGRES_USER: test_user
          POSTGRES_PASSWORD: test_pass
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          
      - name: Install dependencies
        run: |
          cd backend
          pip install -r requirements.txt
          
      - name: Run tests
        env:
          DATABASE_URL: postgresql://test_user:test_pass@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379/0
          SECRET_KEY: test-secret-key
          DEBUG: True
        run: |
          cd backend
          python manage.py test
          
      - name: Run linting
        run: |
          cd backend
          flake8 .
          black --check .
          isort --check-only .

  test-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install dependencies
        run: |
          cd frontend
          npm ci
          
      - name: Run tests
        run: |
          cd frontend
          npm test -- --coverage
          
      - name: Run linting
        run: |
          cd frontend
          npm run lint
          npm run type-check
          
      - name: Build
        env:
          VITE_API_BASE_URL: https://api.bangforyourbuck.ie/api/v1
        run: |
          cd frontend
          npm run build

  deploy-backend:
    needs: [test-backend]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Railway
        uses: bervProject/railway-deploy@main
        with:
          railway_token: ${{ secrets.RAILWAY_TOKEN }}
          service: bangforbuck-api

  deploy-frontend:
    needs: [test-frontend]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          working-directory: ./frontend
          vercel-args: '--prod'

  notify:
    needs: [deploy-backend, deploy-frontend]
    runs-on: ubuntu-latest
    if: always()
    
    steps:
      - name: Send Slack notification
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Deployment ${{ job.status }}'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
        if: always()
GitHub Secrets
Add these secrets to your repository:

Go to Settings > Secrets and variables > Actions
Add:

RAILWAY_TOKEN - From Railway dashboard
VERCEL_TOKEN - From Vercel account settings
VERCEL_ORG_ID - From Vercel project settings
VERCEL_PROJECT_ID - From Vercel project settings
SLACK_WEBHOOK - Slack webhook URL (optional)




Backup Strategy
Database Backups
Automated Daily Backups (DigitalOcean)
Create backup script /home/deploy/scripts/backup-db.sh:
bash#!/bin/bash

# Configuration
BACKUP_DIR="/home/deploy/backups"
DB_NAME="bangforyourbuck"
DB_USER="bangforbuck_user"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/db_backup_$DATE.sql.gz"
S3_BUCKET="s3://bangforbuck-backups"

# Create backup
pg_dump -U $DB_USER $DB_NAME | gzip > $BACKUP_FILE

# Upload to S3 (optional)
aws s3 cp $BACKUP_FILE $S3_BUCKET/

# Keep only last 30 days locally
find $BACKUP_DIR -name "db_backup_*.sql.gz" -mtime +30 -delete

# Send notification
curl -X POST https://hc-ping.com/your-backup-uuid
Make executable:
bashchmod +x /home/deploy/scripts/backup-db.sh
Add to crontab:
bashcrontab -e

# Add:
0 2 * * * /home/deploy/scripts/backup-db.sh
Railway/Render Backups
Railway:

Automatic daily backups included
Download from dashboard: Data > Backups

Render:

Backups available on paid plans
Manual backups via dashboard

Media Backups
Cloudinary stores your media files with redundancy. Additionally:
Download all media (backup script):
python# scripts/backup_media.py
import cloudinary
import cloudinary.api
import os

cloudinary.config(
    cloud_name=os.getenv('CLOUDINARY_CLOUD_NAME'),
    api_key=os.getenv('CLOUDINARY_API_KEY'),
    api_secret=os.getenv('CLOUDINARY_API_SECRET')
)

# Get all resources
resources = cloudinary.api.resources(
    type="upload",
    max_results=500
)

# Download each
for resource in resources['resources']:
    # Download logic here
    pass
Code Backups
Code is backed up in GitHub. Additionally:

Tag releases:

bash   git tag -a v1.0.0 -m "Production release 1.0.0"
   git push origin v1.0.0

Mirror to another service (optional):

GitLab
Bitbucket
AWS CodeCommit




Scaling Considerations
When to Scale
Monitor these metrics:

CPU Usage > 70% sustained
Memory Usage > 80% sustained
Response Time > 500ms average
Error Rate > 1%
Database Connections approaching max

Horizontal Scaling
Backend (Increase Workers)
Railway/Render:

Scale workers via dashboard
Add more Celery workers

DigitalOcean (Docker Compose):
yamlservices:
  backend:
    # ... other config
    deploy:
      replicas: 3  # Run 3 instances
Or use Docker Swarm/Kubernetes.
Database Read Replicas
For read-heavy loads:

Set up PostgreSQL read replica
Configure Django database routing:

python# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.contrib.gis.db.backends.postgis',
        'NAME': 'bangforyourbuck',
        # ... primary database config
    },
    'replica': {
        'ENGINE': 'django.contrib.gis.db.backends.postgis',
        'NAME': 'bangforyourbuck',
        # ... replica database config
    }
}

# Database router
DATABASE_ROUTERS = ['config.db_router.PrimaryReplicaRouter']
CDN for Frontend
Already included with Vercel/Netlify. For custom setup:

Use Cloudflare CDN (free tier)
Cache static assets aggressively
Enable Brotli compression

Vertical Scaling
Upgrade server resources:

More CPU cores
More RAM
Faster disk (SSD)

Railway/Render: Upgrade plan in dashboard
DigitalOcean: Resize droplet via dashboard or CLI
Caching Strategy
Redis Caching
Already configured. Adjust cache times in views:
pythonfrom django.views.decorators.cache import cache_page
from django.core.cache import cache

# Cache view for 5 minutes
@cache_page(60 * 5)
def venue_list(request):
    pass

# Manual caching
def get_stats():
    stats = cache.get('homepage_stats')
    if not stats:
        stats = calculate_stats()
        cache.set('homepage_stats', stats, 60 * 5)  # 5 minutes
    return stats
Database Query Optimization
python# Use select_related for foreign keys
venues = Venue.objects.select_related('county').all()

# Use prefetch_related for many-to-many
venues = Venue.objects.prefetch_related('prices').all()

# Add database indexes
class Venue(models.Model):
    name = models.CharField(max_length=200, db_index=True)
    county = models.CharField(max_length=50, db_index=True)
    
    class Meta:
        indexes = [
            models.Index(fields=['county', 'is_active']),
            models.Index(fields=['-created_at']),
        ]

Troubleshooting
Common Issues
Issue: 502 Bad Gateway
Possible Causes:

Backend not running
Database connection failed
Out of memory

Solutions:
bash# Check backend status
railway status  # Railway
docker-compose ps  # Docker

# Check logs
railway logs  # Railway
docker-compose logs backend  # Docker

# Restart service
railway restart  # Railway
docker-compose restart backend  # Docker
Issue: Static Files Not Loading
Solution:
bash# Collect static files
python manage.py collectstatic --noinput

# Verify STATIC_ROOT setting
# Verify nginx/web server configuration
Issue: Database Connection Errors
Check:
bash# Test connection
psql -U bangforbuck_user -h localhost -d bangforyourbuck

# Check DATABASE_URL format
echo $DATABASE_URL

# Verify PostgreSQL is running
sudo systemctl status postgresql
Issue: Celery Tasks Not Running
Check:
bash# Verify Celery worker is running
ps aux | grep celery

# Check Redis connection
redis-cli ping

# Restart worker
celery -A config worker --loglevel=info

# Check task queue
from celery import current_app
current_app.control.inspect().active()
Issue: High Memory Usage
Solutions:

Check for memory leaks in code
Reduce number of workers
Add swap space (temporary)
Upgrade server RAM

bash# Add swap (DigitalOcean)
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
Issue: Slow Database Queries
Diagnose:
sql-- Find slow queries
SELECT pid, now() - pg_stat_activity.query_start AS duration, query 
FROM pg_stat_activity 
WHERE (now() - pg_stat_activity.query_start) > interval '5 seconds';

-- Check table sizes
SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
Solutions:

Add indexes
Optimize queries (use select_related, prefetch_related)
Archive old data
Run VACUUM ANALYZE

Issue: API Rate Limiting
Adjust rate limits in settings.py:
pythonREST_FRAMEWORK = {
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/minute',  # Increase if needed
        'user': '1000/minute',
        'submission': '10/day',
    }
}
Emergency Rollback
If deployment causes issues:
Railway:
bashrailway rollback
Vercel:
bashvercel rollback
DigitalOcean (Docker):
bash# Checkout previous version
git checkout v1.0.0

# Rebuild and restart
docker-compose -f docker-compose.prod.yml up -d --build
Health Check Endpoint
Create health check for monitoring:
python# apps/core/views.py
from django.http import JsonResponse
from django.db import connection
from django.core.cache import cache

def health_check(request):
    # Check database
    try:
        with connection.cursor() as cursor:
            cursor.execute("SELECT 1")
    except Exception as e:
        return JsonResponse({'status': 'unhealthy', 'database': str(e)}, status=503)
    
    # Check Redis
    try:
        cache.set('health_check', 'ok', 10)
        if cache.get('health_check') != 'ok':
            raise Exception("Cache not working")
    except Exception as e:
        return JsonResponse({'status': 'unhealthy', 'cache': str(e)}, status=503)
    
    return JsonResponse({
        'status': 'healthy',
        'database': 'ok',
        'cache': 'ok'
    })

# urls.py
urlpatterns = [
    path('api/health/', health_check),
]

Post-Deployment Checklist
After deploying to production:

 Verify all environment variables are set correctly
 Test user registration and login
 Submit a test price and verify it appears
 Test map functionality and venue search
 Check that images upload correctly
 Verify email sending works
 Test mobile responsiveness
 Check SSL certificate is valid
 Verify error tracking is working (Sentry)
 Test all API endpoints
 Set up uptime monitoring
 Configure database backups
 Set up log rotation
 Test error pages (404, 500)
 Verify analytics are tracking
 Check CORS configuration
 Test rate limiting
 Verify security headers
 Run security scan (OWASP ZAP, Snyk)
 Update documentation with production URLs


Maintenance Schedule
Daily

Monitor error rates (Sentry)
Check uptime (UptimeRobot)
Review application logs

Weekly

Review slow query log
Check disk space usage
Review backup success
Update dependencies (security patches)

Monthly

Full database backup verification
Review and archive old data
Performance testing
Security audit
Update all dependencies

Quarterly

Disaster recovery drill
Review and update documentation
Capacity planning review
Cost optimization review


Support & Resources

Production Dashboard: [Railway/Render Dashboard URL]
Status Page: https://status.bangforyourbuck.ie
Sentry: [Your Sentry project URL]
Cloudinary: https://cloudinary.com/console
Documentation: https://docs.bangforyourbuck.ie

Emergency Contacts:

Technical Lead: tech@bangforyourbuck.ie
On-Call Phone: [Your number]


Last Updated: October 14, 2024
Deployment Version: 1.0.0

🎉 Congratulations! Your application is now live in production. Monitor closely for the first 24-48 hours and be ready to respond to issues quickly. Good luck!

---

All three supplementary documentation files are now complete! You have:

1. **API.md** - Complete API documentation with all endpoints, authentication, rate limiting, error handling, and examples
2. **CONTRIBUTING.md** - Comprehensive contribution guidelines covering code style, commit conventions, PR process, and community guidelines
3. **DEPLOYMENT.md** - Detailed deployment guide for Railway, Render, DigitalOcean, Vercel, and Netlify with monitoring, backup, scaling, and troubleshooting

These docs should give you everything you need to build, maintain, and scale Bang For Your Buck! Let me know if you need any clarifications or additional documentation.

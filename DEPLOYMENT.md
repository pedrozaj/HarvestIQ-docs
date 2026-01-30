# HarvestIQ Deployment Guide

## Overview

| Component | Platform | Trigger |
|-----------|----------|---------|
| Frontend | Vercel | Push to `main` branch |
| Backend | Railway | Push to `main` branch |
| Database | Railway PostgreSQL | Managed service |
| Storage | Cloudflare R2 | Via backend API |

---

## Frontend Deployment (Vercel)

### Automatic Deployment
Push to GitHub and Vercel automatically deploys:

```bash
cd HarvestIQ
git add .
git commit -m "Your changes"
git push
```

### Manual Deployment
```bash
# Install Vercel CLI
npm install -g vercel

# Login (first time)
vercel login

# Deploy to production
vercel --prod
```

### Vercel Dashboard
- URL: https://vercel.com/joeys-projects-7189f482/harvest-iq

---

## Backend Deployment (Railway)

Railway hosts three services: **HarvestIQ-backend** (API server), **HarvestIQ-worker** (background job processor), and **Postgres** (database).

### Automatic Deployment
Push to GitHub and Railway automatically deploys both backend and worker:

```bash
cd HarvestIQ-backend
git add .
git commit -m "Your changes"
git push
```

### Railway CLI Setup
```bash
# Install Railway CLI (macOS)
brew install railway

# Login via browser
railway login
```

### Railway CLI Commands

#### Project Management
```bash
# List all projects
railway list

# Link to HarvestIQ project
railway link -p 1e5eb883-85fd-4867-8603-050dbb3fd6f1

# Check project status
railway status
```

#### Deployment
```bash
# Deploy current directory
railway up

# View deployment logs
railway logs
```

### Railway Dashboard
- URL: https://railway.com/project/1e5eb883-85fd-4867-8603-050dbb3fd6f1

### Environment Variables

Both the backend and worker services require environment variables. When adding a new variable, ensure it is added to **both services** if needed.

#### HarvestIQ-backend (API Server)

| Variable | Purpose |
|----------|---------|
| `DATABASE_URL` | PostgreSQL connection (auto-linked) |
| `JWT_SECRET` | Authentication token signing |
| `JWT_ACCESS_EXPIRY` | Access token lifetime (default: 15m) |
| `JWT_REFRESH_EXPIRY` | Refresh token lifetime (default: 7d) |
| `NODE_ENV` | Environment (production) |
| `FRONTEND_URL` | Frontend URL for CORS and email links |
| `RESEND_API_KEY` | Resend email API key |
| `EMAIL_FROM` | From address for outbound emails |
| `EMAIL_INBOUND_DOMAIN` | Domain for inbound email parsing |
| `ANTHROPIC_API_KEY` | Claude API for AI features |
| `OPENAI_API_KEY` | OpenAI API for embeddings |
| `FRED_API_KEY` | FRED API for housing data |
| `R2_ACCOUNT_ID` | Cloudflare R2 account |
| `R2_ACCESS_KEY_ID` | R2 access key |
| `R2_SECRET_ACCESS_KEY` | R2 secret key |
| `R2_BUCKET_NAME` | R2 bucket name |
| `PUPPETEER_SKIP_CHROMIUM_DOWNLOAD` | Puppeteer config |

#### HarvestIQ-worker (Background Jobs)

The worker processes inbound emails, sends reply confirmations, and runs background tasks. It needs most of the same variables as the backend.

| Variable | Purpose |
|----------|---------|
| `DATABASE_URL` | PostgreSQL connection (auto-linked) |
| `JWT_SECRET` | Shared with backend for token validation |
| `NODE_ENV` | Environment (production) |
| `FRONTEND_URL` | Used in email links (e.g., "View in HarvestIQ") |
| `RESEND_API_KEY` | Sends reply confirmation emails |
| `EMAIL_FROM` | From address on reply emails |
| `EMAIL_INBOUND_DOMAIN` | Domain for reply-to addresses |
| `ANTHROPIC_API_KEY` | AI classification of inbound emails |
| `OPENAI_API_KEY` | Embeddings for document search |
| `R2_ACCOUNT_ID` | Stores email attachments/PDFs |
| `R2_ACCESS_KEY_ID` | R2 access key |
| `R2_SECRET_ACCESS_KEY` | R2 secret key |
| `R2_BUCKET_NAME` | R2 bucket name |
| `PUPPETEER_SKIP_CHROMIUM_DOWNLOAD` | Puppeteer config |

> **Important:** If a variable is missing from the worker, it falls back to code defaults (e.g., `FRONTEND_URL` defaults to `http://localhost:3000`), which causes incorrect links in emails sent from the worker.

---

## Database Management (PostgreSQL)

### Connection Details
- **Host:** shortline.proxy.rlwy.net
- **Port:** 19686
- **Database:** railway
- **Username:** postgres

### Connect via psql
```bash
# Install psql client (macOS)
brew install libpq
echo 'export PATH="/opt/homebrew/opt/libpq/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Connect to database
psql "$DATABASE_URL"
# Get DATABASE_URL from SENSITIVE-INFO.md or Railway dashboard
```

### Common SQL Commands
```sql
-- List all tables
\dt

-- View settings
SELECT * FROM settings;

-- Update a setting
UPDATE settings SET value = 'New Value' WHERE key = 'site_title';

-- View all files
SELECT id, original_name, size, created_at FROM files ORDER BY created_at DESC;

-- Delete a file record (also delete from R2 via API)
DELETE FROM files WHERE id = 1;
```

### Database Schema
```sql
-- Settings table
CREATE TABLE settings (
  id SERIAL PRIMARY KEY,
  key TEXT UNIQUE NOT NULL,
  value TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
);

-- Files table
CREATE TABLE files (
  id SERIAL PRIMARY KEY,
  filename TEXT NOT NULL,
  original_name TEXT NOT NULL,
  mime_type TEXT NOT NULL,
  size INTEGER NOT NULL,
  storage_key TEXT UNIQUE NOT NULL,
  project_id INTEGER,
  uploaded_by INTEGER,
  created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
);
```

---

## File Storage (Cloudflare R2)

Files are managed through the backend API, which proxies to R2:

### Upload File
```bash
curl -X POST https://harvestiq-backend-production.up.railway.app/api/files \
  -F "file=@/path/to/document.pdf"
```

### Download File
```bash
curl -O https://harvestiq-backend-production.up.railway.app/api/files/1/download
```

### Delete File
```bash
curl -X DELETE https://harvestiq-backend-production.up.railway.app/api/files/1
```

### R2 Dashboard
- URL: https://dash.cloudflare.com/af94cdfdfcd7e6bb0a27faa94579ea7e

---

## Deployment Checklist

### Before Deploying Frontend
- [ ] Test locally with `npm run dev`
- [ ] Run `npm run build` to check for build errors
- [ ] Verify API_URL points to production backend
- [ ] Commit all changes

### Before Deploying Backend
- [ ] Test locally with `npm run dev`
- [ ] Run `npm run build` to compile TypeScript
- [ ] Test compiled version with `npm start`
- [ ] Commit all changes including `package-lock.json`

---

## Rollback Procedures

### Vercel
1. Go to Deployments tab
2. Find previous working deployment
3. Click "..." menu → "Promote to Production"

### Railway
1. Go to Deployments tab
2. Click on previous deployment
3. Click "Rollback"

---

## Monitoring

### Vercel
- Build logs: Dashboard → Deployments → Click deployment
- Runtime logs: Dashboard → Logs tab

### Railway
- Logs: Dashboard → Click service → Logs
- Metrics: Dashboard → Click service → Metrics
- Database: Dashboard → Postgres service → Data tab

### Cloudflare R2
- Storage metrics: Dashboard → R2 → harvestiq-files bucket

---

## Troubleshooting

### Backend Not Starting
```bash
# Check Railway logs
railway logs

# Common issues:
# - Missing DATABASE_URL (ensure Postgres service is linked)
# - Missing R2 credentials (check environment variables)
```

### Database Connection Issues
```bash
# Test connection
psql "postgresql://postgres:PASSWORD@shortline.proxy.rlwy.net:19686/railway" -c "SELECT 1"

# Check if tables exist
psql "..." -c "\dt"
```

### File Upload Failures
```bash
# Check if R2 credentials are set
railway variables | grep R2

# Test R2 connectivity (via API)
curl https://harvestiq-backend-production.up.railway.app/api/files
```


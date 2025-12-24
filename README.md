# HarvestIQ Documentation

This directory contains documentation for the HarvestIQ construction management platform.

## Project Overview

HarvestIQ is a construction management platform that helps builders track expenses, manage contractors, and keep projects on schedule.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         HarvestIQ System                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐     ┌─────────────────┐     ┌───────────┐ │
│  │    Frontend     │────▶│   Backend API   │────▶│ PostgreSQL│ │
│  │    (Vercel)     │     │   (Railway)     │     │ (Railway) │ │
│  │   Next.js 15    │     │  Express + TS   │     │           │ │
│  └─────────────────┘     └────────┬────────┘     └───────────┘ │
│                                   │                             │
│                                   ▼                             │
│                          ┌─────────────────┐                    │
│                          │   Cloudflare    │                    │
│                          │   R2 Storage    │                    │
│                          └─────────────────┘                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Repositories

| Repository | Description | Deployment |
|------------|-------------|------------|
| HarvestIQ | Next.js frontend application | Vercel |
| HarvestIQ-backend | Express.js API with PostgreSQL | Railway |

## Documentation Files

| File | Description |
|------|-------------|
| README.md | This file - project overview |
| ARCHITECTURE.md | Detailed system architecture |
| SETUP.md | Development environment setup |
| DEPLOYMENT.md | Deployment procedures |
| API.md | API documentation |
| SENSITIVE-INFO.md | **PRIVATE** - Credentials and secrets |

## Quick Links

- Frontend: https://harvest-iq-rosy.vercel.app
- Backend API: https://harvestiq-backend-production.up.railway.app

## Tech Stack

### Frontend
- Next.js 15
- React 19
- TypeScript
- Tailwind CSS

### Backend
- Express.js 5
- TypeScript
- PostgreSQL
- Cloudflare R2 (S3-compatible storage)

### Infrastructure
- Vercel (Frontend hosting)
- Railway (Backend + PostgreSQL hosting)
- Cloudflare R2 (Object storage)
- GitHub (Source control)


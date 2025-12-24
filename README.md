# HarvestIQ Documentation

This directory contains documentation for the HarvestIQ construction management platform.

## Project Overview

HarvestIQ is a multi-tenant construction management platform that helps builders track expenses, manage contractors, and keep projects on schedule.

## Current Status

**Phase 1 Complete:** Infrastructure & Authentication
- Multi-tenant architecture (builders, users, organizations)
- Full authentication flow (register, verify, login, password reset)
- User profile & session management
- Email integration (Resend)
- Rate limiting & security

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         HarvestIQ System                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐     ┌─────────────────┐     ┌───────────┐ │
│  │    Frontend     │────▶│   Backend API   │────▶│ PostgreSQL│ │
│  │    (Vercel)     │     │   (Railway)     │     │ (Railway) │ │
│  │   Next.js 15    │     │  Express 5 + TS │     │           │ │
│  └─────────────────┘     └────────┬────────┘     └───────────┘ │
│                                   │                             │
│                          ┌────────┴────────┐                    │
│                          │                 │                    │
│                          ▼                 ▼                    │
│                 ┌─────────────────┐ ┌─────────────────┐         │
│                 │   Cloudflare    │ │     Resend      │         │
│                 │   R2 Storage    │ │  (Email API)    │         │
│                 └─────────────────┘ └─────────────────┘         │
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
| ARCHITECTURE.md | System architecture & database schema |
| SETUP.md | Development environment setup |
| DEPLOYMENT.md | Deployment procedures |
| API.md | API endpoint documentation |
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
- shadcn/ui components
- React Hook Form + Zod

### Backend
- Express.js 5
- TypeScript
- PostgreSQL
- JWT authentication
- Resend (email)
- Cloudflare R2 (storage)

### Infrastructure
- Vercel (Frontend hosting)
- Railway (Backend + PostgreSQL)
- Cloudflare R2 (Object storage)
- Resend (Transactional email)
- GitHub (Source control)

## Multi-Tenant Model

```
Builder (Company)
    ├── Users (Individual accounts)
    ├── Organizations (Business units)
    └── Organization Members (Role assignments)
```

Each **Builder** is an isolated tenant with its own:
- Subscription & billing
- Storage quotas
- Users and organizations
- All associated data

# HarvestIQ Architecture

## System Components

### 1. Frontend (HarvestIQ)

**Technology:** Next.js 15 with React 19 and TypeScript

**Hosting:** Vercel (automatic deployments from GitHub)

**Features:**
- Server-side rendering for SEO
- Client-side interactivity for dashboard
- File upload/download via backend API
- Settings loaded from backend API

**Directory Structure:**
```
HarvestIQ/
├── src/
│   ├── app/
│   │   ├── page.tsx          # Landing page
│   │   ├── layout.tsx        # Root layout
│   │   ├── globals.css       # Global styles
│   │   ├── dashboard/        # Dashboard page
│   │   ├── documents/        # File management
│   │   ├── projects/         # Project tracking (UI)
│   │   ├── expenses/         # Expense management (UI)
│   │   ├── contractors/      # Contractor management (UI)
│   │   └── schedule/         # Scheduling (UI)
│   └── components/
│       ├── Navbar.tsx
│       ├── DashboardLayout.tsx
│       └── FeatureCard.tsx
├── public/
├── package.json
└── tsconfig.json
```

### 2. Backend API (HarvestIQ-backend)

**Technology:** Express.js 5 with TypeScript

**Hosting:** Railway (automatic deployments from GitHub)

**Database:** PostgreSQL (Railway managed, persistent volume)

**Storage:** Cloudflare R2 via S3-compatible API

**Features:**
- RESTful API endpoints
- Settings management (CRUD)
- File management (upload, download, delete via R2)
- CORS enabled for frontend access
- Auto-seeding database on startup

**Directory Structure:**
```
HarvestIQ-backend/
├── src/
│   ├── index.ts              # Express server
│   ├── db.ts                 # PostgreSQL database setup
│   └── r2.ts                 # R2 storage client (S3 SDK)
├── dist/                     # Compiled JavaScript
├── package.json
└── tsconfig.json
```

## Data Flow

### 1. Page Load (Landing Page)
```
User Browser → Vercel → Next.js SSR → Railway API → PostgreSQL
                ↓
           Render page with site settings
```

### 2. File Upload
```
User Browser → Vercel (Documents Page) → Railway API → R2 Storage
                                              ↓
                                    Save metadata to PostgreSQL
                                              ↓
                                        Return file record
```

### 3. File Download
```
User Browser → Railway API → R2 Storage → Stream file to browser
```

## Database Schema

### settings table
| Column | Type | Description |
|--------|------|-------------|
| id | SERIAL | Primary key |
| key | TEXT | Unique setting key |
| value | TEXT | Setting value |
| created_at | TIMESTAMPTZ | Creation timestamp |
| updated_at | TIMESTAMPTZ | Last update timestamp |

### files table
| Column | Type | Description |
|--------|------|-------------|
| id | SERIAL | Primary key |
| filename | TEXT | Storage filename |
| original_name | TEXT | Original filename |
| mime_type | TEXT | File MIME type |
| size | INTEGER | File size in bytes |
| storage_key | TEXT | R2 storage key |
| project_id | INTEGER | Project association (nullable) |
| uploaded_by | INTEGER | User association (nullable) |
| created_at | TIMESTAMPTZ | Upload timestamp |
| updated_at | TIMESTAMPTZ | Last update timestamp |

## Infrastructure

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


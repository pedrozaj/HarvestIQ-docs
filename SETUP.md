# HarvestIQ Development Setup

## Prerequisites

- Node.js 18+ (recommended: 20 or 22)
- npm or yarn
- Git
- Railway CLI (for backend management)
- psql client (for database access)

## Clone Repositories

```bash
cd /Users/joey/Projects/X1LLC

# Frontend
git clone git@github.com:pedrozaj/HarvestIQ.git

# Backend
git clone git@github.com:pedrozaj/HarvestIQ-backend.git
```

## Backend Setup

```bash
cd HarvestIQ-backend

# Install dependencies
npm install

# Create .env file for local development
cat > .env << 'EOF'
# Server
NODE_ENV=development
PORT=3001

# Database
DATABASE_URL=<see SENSITIVE-INFO.md>

# JWT
JWT_SECRET=harvestiq-dev-secret-key-min-32-characters-long
JWT_ACCESS_EXPIRY=15m
JWT_REFRESH_EXPIRY=7d

# Frontend URL (for CORS and email links)
FRONTEND_URL=http://localhost:3000

# Email (Resend) - optional for local dev
RESEND_API_KEY=<see SENSITIVE-INFO.md>
EMAIL_FROM=HarvestIQ <noreply@harvestiq.thex1.com>

# R2 Storage - optional for local dev
R2_ACCOUNT_ID=<see SENSITIVE-INFO.md>
R2_ACCESS_KEY_ID=<see SENSITIVE-INFO.md>
R2_SECRET_ACCESS_KEY=<see SENSITIVE-INFO.md>
R2_BUCKET_NAME=harvestiq-files
EOF

# Run database migrations
npm run migrate

# Run in development mode (with hot reload)
npm run dev

# Server runs on http://localhost:3001
```

## Frontend Setup

```bash
cd HarvestIQ

# Install dependencies
npm install

# Create .env.local for local development
cat > .env.local << 'EOF'
NEXT_PUBLIC_API_URL=http://localhost:3001
EOF

# Run development server
npm run dev

# Open http://localhost:3000
```

## CLI Tools Setup

### Railway CLI
```bash
# Install (macOS)
brew install railway

# Login via browser
railway login

# Link to project
railway link -p 1e5eb883-85fd-4867-8603-050dbb3fd6f1
```

### PostgreSQL Client
```bash
# Install psql client (macOS)
brew install libpq

# Add to PATH
echo 'export PATH="/opt/homebrew/opt/libpq/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Test connection
psql "$DATABASE_URL" -c "SELECT 1"
# Get DATABASE_URL from SENSITIVE-INFO.md
```

## Testing the Setup

### Test Backend API
```bash
# Health check
curl http://localhost:3001/health

# API info
curl http://localhost:3001
```

### Test Authentication Flow
```bash
# Register a new account
curl -X POST http://localhost:3001/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "company_name": "Test Company",
    "name": "Test User",
    "email": "test@example.com",
    "phone": "555-0123",
    "password": "TestPass123"
  }'

# Check backend console for verification email (logged when Resend not configured)

# Login (after verifying email)
curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "TestPass123"
  }'
```

### Test Protected Endpoints
```bash
# Get current user (requires auth cookie or token)
curl http://localhost:3001/api/users/me \
  -H "Authorization: Bearer <access_token>"

# Get builder info
curl http://localhost:3001/api/builder \
  -H "Authorization: Bearer <access_token>"
```

### Test Production API
```bash
# Health check
curl https://harvestiq-backend-production.up.railway.app/health

# API info
curl https://harvestiq-backend-production.up.railway.app
```

## Project Structure

```
/Users/joey/Projects/X1LLC/
├── HarvestIQ/                  # Frontend (Next.js 15)
│   ├── src/
│   │   ├── app/                # App router pages
│   │   │   ├── (auth)/         # Auth pages (login, register, etc.)
│   │   │   ├── dashboard/      # Dashboard
│   │   │   └── ...             # Feature pages
│   │   ├── components/         # React components
│   │   ├── hooks/              # Custom hooks (useAuth, etc.)
│   │   ├── lib/                # API client, utilities
│   │   └── schemas/            # Zod validation schemas
│   └── package.json
├── HarvestIQ-backend/          # Backend (Express 5)
│   ├── src/
│   │   ├── config/             # Database, environment
│   │   ├── controllers/        # Route handlers
│   │   ├── middleware/         # Auth, validation, rate limiting
│   │   ├── models/             # Database models
│   │   ├── routes/             # API routes
│   │   ├── schemas/            # Zod schemas
│   │   ├── services/           # Email, etc.
│   │   └── utils/              # JWT, crypto helpers
│   ├── migrations/             # SQL migrations
│   └── package.json
└── HarvestIQ-docs/             # Documentation
    ├── README.md               # Project overview
    ├── ARCHITECTURE.md         # System design
    ├── API.md                  # API reference
    ├── SETUP.md                # This file
    ├── DEPLOYMENT.md           # Deploy guide
    └── SENSITIVE-INFO.md       # Credentials
```

## Database Migrations

```bash
cd HarvestIQ-backend

# Run all pending migrations
npm run migrate

# View migration files
ls migrations/
```

## Common Issues

### Port Already in Use
```bash
# Find process using port
lsof -i :3000  # or :3001

# Kill it
kill -9 <PID>
```

### Database Connection Failed
```bash
# Check if Railway is accessible
psql "$DATABASE_URL" -c "\dt"

# Common causes:
# - VPN blocking connection
# - Firewall rules
# - Invalid credentials
```

### Email Not Sending
- If `RESEND_API_KEY` is not set, emails are logged to console
- Check backend console for verification links during local development
- Ensure domain is verified in Resend dashboard for production

### JWT Errors
```bash
# Ensure JWT_SECRET is at least 32 characters
# Check that JWT_ACCESS_EXPIRY and JWT_REFRESH_EXPIRY are valid duration strings
```

### Git SSH Issues
```bash
# Test SSH connection
ssh -T git@github.com

# If failed, ensure SSH key is added
ssh-add ~/.ssh/id_ed25519
```

### Node Version Issues
```bash
# Check node version
node --version

# Use nvm to switch versions
nvm use 20
```

### Railway CLI Issues
```bash
# Re-login if session expired
railway logout
railway login

# Re-link if project changed
railway unlink
railway link -p 1e5eb883-85fd-4867-8603-050dbb3fd6f1
```

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

## Frontend Setup

```bash
cd HarvestIQ

# Install dependencies
npm install

# Run development server
npm run dev

# Open http://localhost:3000
```

## Backend Setup

```bash
cd HarvestIQ-backend

# Install dependencies
npm install

# Create .env file for local development
# Get credentials from SENSITIVE-INFO.md (never commit this file)
cat > .env << 'EOF'
PORT=3001
NODE_ENV=development
DATABASE_URL=<see SENSITIVE-INFO.md>
R2_ACCOUNT_ID=<see SENSITIVE-INFO.md>
R2_ACCESS_KEY_ID=<see SENSITIVE-INFO.md>
R2_SECRET_ACCESS_KEY=<see SENSITIVE-INFO.md>
R2_BUCKET_NAME=harvestiq-files
EOF

# Build TypeScript
npm run build

# Run server
npm start

# Or run in development mode (with hot reload)
npm run dev

# Server runs on http://localhost:3001
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
curl http://localhost:3001

# Get settings
curl http://localhost:3001/api/settings

# List files
curl http://localhost:3001/api/files
```

### Test Production API
```bash
# Health check
curl https://harvestiq-backend-production.up.railway.app

# Get settings
curl https://harvestiq-backend-production.up.railway.app/api/settings
```

### Test File Upload
```bash
# Upload a file
curl -X POST http://localhost:3001/api/files \
  -F "file=@/path/to/test.pdf"
```

## Project Structure

```
/Users/joey/Projects/X1LLC/
├── HarvestIQ/              # Frontend (Next.js 15)
│   ├── src/
│   │   ├── app/            # App router pages
│   │   └── components/     # React components
│   ├── public/
│   └── package.json
├── HarvestIQ-backend/      # Backend (Express 5)
│   ├── src/
│   │   ├── index.ts        # Express server
│   │   ├── db.ts           # PostgreSQL client
│   │   └── r2.ts           # R2 storage client
│   ├── dist/               # Compiled JS
│   └── package.json
└── HarvestIQ-docs/         # Documentation
    ├── README.md           # Project overview
    ├── ARCHITECTURE.md     # System design
    ├── API.md              # API reference
    ├── SETUP.md            # This file
    ├── DEPLOYMENT.md       # Deploy guide
    └── SENSITIVE-INFO.md   # Credentials
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


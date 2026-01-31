# HarvestIQ Architecture

## System Components

### 1. Frontend (HarvestIQ)

**Technology:** Next.js 15 with React 19 and TypeScript

**Hosting:** Vercel (automatic deployments from GitHub)

**Features:**
- Server-side rendering for SEO
- Client-side interactivity for dashboard
- Cookie-based JWT authentication
- React Hook Form with Zod validation

**Directory Structure:**
```
HarvestIQ/
├── src/
│   ├── app/
│   │   ├── page.tsx              # Landing page
│   │   ├── layout.tsx            # Root layout
│   │   ├── globals.css           # Global styles
│   │   ├── (auth)/               # Auth pages (grouped layout)
│   │   │   ├── layout.tsx        # Auth layout (centered card)
│   │   │   ├── login/page.tsx
│   │   │   ├── register/page.tsx
│   │   │   ├── verify-email/page.tsx
│   │   │   ├── forgot-password/page.tsx
│   │   │   └── reset-password/page.tsx
│   │   ├── dashboard/            # Dashboard page
│   │   ├── documents/            # File management
│   │   ├── projects/             # Project tracking
│   │   ├── expenses/             # Expense management
│   │   ├── contractors/          # Contractor management
│   │   └── schedule/             # Scheduling
│   ├── components/
│   │   ├── ui/                   # shadcn/ui components
│   │   ├── Navbar.tsx
│   │   └── DashboardLayout.tsx
│   ├── hooks/
│   │   └── useAuth.ts            # Auth context & hooks
│   ├── lib/
│   │   ├── api.ts                # API client
│   │   └── constants.ts          # Error messages
│   └── schemas/
│       └── auth.schema.ts        # Zod validation schemas
├── public/
├── package.json
└── tsconfig.json
```

### 2. Backend API (HarvestIQ-backend)

**Technology:** Express.js 5 with TypeScript

**Hosting:** Railway (automatic deployments from GitHub)

**Database:** PostgreSQL (Railway managed)

**Storage:** Cloudflare R2 via S3-compatible API

**Email:** Resend (transactional emails)

**Features:**
- RESTful API endpoints
- Multi-tenant architecture (builder isolation)
- JWT authentication with refresh tokens
- Rate limiting per endpoint
- Helmet security headers

**Directory Structure:**
```
HarvestIQ-backend/
├── src/
│   ├── index.ts                  # Server entry point
│   ├── app.ts                    # Express app setup
│   ├── config/
│   │   ├── database.ts           # PostgreSQL pool + transactions
│   │   └── env.ts                # Environment validation (Zod)
│   ├── constants/
│   │   └── index.ts              # Enums, error codes
│   ├── controllers/
│   │   ├── auth.controller.ts
│   │   ├── user.controller.ts
│   │   ├── builder.controller.ts
│   │   ├── organization.controller.ts
│   │   ├── project.controller.ts
│   │   ├── activity.controller.ts
│   │   └── invitation.controller.ts
│   ├── middleware/
│   │   ├── auth.ts               # JWT verification
│   │   ├── errorHandler.ts       # Global error handler
│   │   ├── rateLimiter.ts        # Rate limiting
│   │   └── validate.ts           # Zod validation
│   ├── models/
│   │   ├── user.model.ts
│   │   ├── builder.model.ts
│   │   ├── organization.model.ts
│   │   ├── organizationMember.model.ts
│   │   ├── refreshToken.model.ts
│   │   ├── project.model.ts
│   │   ├── activity.model.ts
│   │   └── invitation.model.ts
│   ├── routes/
│   │   ├── index.ts              # Route aggregator
│   │   ├── auth.routes.ts
│   │   ├── user.routes.ts
│   │   ├── builder.routes.ts
│   │   ├── organization.routes.ts
│   │   ├── project.routes.ts
│   │   ├── activity.routes.ts
│   │   └── invitation.routes.ts
│   ├── schemas/
│   │   ├── auth.schema.ts
│   │   ├── user.schema.ts
│   │   ├── builder.schema.ts
│   │   ├── organization.schema.ts
│   │   ├── project.schema.ts
│   │   └── activity.schema.ts
│   ├── services/
│   │   └── email.service.ts      # Resend email sending
│   ├── types/
│   │   └── index.ts              # TypeScript interfaces
│   ├── utils/
│   │   ├── jwt.ts                # Token generation
│   │   ├── crypto.ts             # Hashing utilities
│   │   ├── dates.ts              # Date helpers
│   │   ├── response.ts           # HTTP response helpers
│   │   ├── scheduleTemplates.ts  # Default schedule generation
│   │   └── budgetTemplates.ts    # Default budget generation
│   ├── db.ts                     # Database query helpers
│   └── r2.ts                     # R2 storage client
├── migrations/
│   ├── 001_create_builders.sql
│   ├── 002_create_users.sql
│   ├── 003_create_organizations.sql
│   ├── 004_create_organization_members.sql
│   ├── 005_create_refresh_tokens.sql
│   ├── 006_create_projects.sql
│   ├── 007_create_activity_log.sql
│   ├── 008_create_invitations.sql
│   └── 009_add_invitations_deleted_at.sql
├── package.json
└── tsconfig.json
```

## Multi-Tenant Architecture

HarvestIQ uses a multi-tenant architecture where each **Builder** (company) has isolated data.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Multi-Tenant Hierarchy                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                          BUILDER                                 │   │
│  │  (Tenant - Company Account)                                      │   │
│  │  • Subscription management                                       │   │
│  │  • Storage/AI token limits                                       │   │
│  │  • Billing info                                                  │   │
│  └──────────────────────────┬──────────────────────────────────────┘   │
│                             │                                           │
│              ┌──────────────┴──────────────┐                           │
│              │                             │                           │
│              ▼                             ▼                           │
│  ┌───────────────────────┐   ┌───────────────────────┐                 │
│  │        USERS          │   │    ORGANIZATIONS      │                 │
│  │  (Individual accounts)│   │   (Business units)    │                 │
│  │  • Authentication     │   │  • Project grouping   │                 │
│  │  • Preferences        │   │  • Team separation    │                 │
│  └───────────┬───────────┘   └───────────┬───────────┘                 │
│              │                           │                             │
│              └───────────┬───────────────┘                             │
│                          │                                             │
│                          ▼                                             │
│              ┌───────────────────────┐                                 │
│              │  ORGANIZATION_MEMBERS │                                 │
│              │  (Role assignments)   │                                 │
│              │  • admin              │                                 │
│              │  • manager            │                                 │
│              │  • member             │                                 │
│              │  • viewer             │                                 │
│              └───────────────────────┘                                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Database Schema

### builders table
| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| name | VARCHAR(255) | Company name |
| subscription_status | VARCHAR(50) | trial, active, past_due, canceled |
| subscription_plan | VARCHAR(50) | free, starter, professional, enterprise |
| trial_ends_at | TIMESTAMPTZ | Trial expiration date |
| billing_email | VARCHAR(255) | Billing contact email |
| storage_used_bytes | BIGINT | Current storage usage |
| storage_limit_bytes | BIGINT | Storage quota (default 5GB) |
| ai_tokens_used_month | INTEGER | AI tokens used this month |
| ai_tokens_limit_month | INTEGER | Monthly AI token limit |
| settings | JSONB | Builder-specific settings |
| created_at | TIMESTAMPTZ | Creation timestamp |
| updated_at | TIMESTAMPTZ | Last update timestamp |
| deleted_at | TIMESTAMPTZ | Soft delete timestamp |

### users table
| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| builder_id | UUID | FK to builders |
| email | VARCHAR(255) | Unique email address |
| password_hash | VARCHAR(255) | Bcrypt hash |
| name | VARCHAR(255) | Display name |
| phone | VARCHAR(50) | Phone number |
| timezone | VARCHAR(50) | User timezone |
| notification_preferences | JSONB | Notification settings |
| email_verified_at | TIMESTAMPTZ | Email verification date |
| email_verification_token | VARCHAR(255) | Pending verification token |
| email_verification_expires_at | TIMESTAMPTZ | Token expiration |
| last_login_at | TIMESTAMPTZ | Last successful login |
| failed_login_attempts | INTEGER | Failed login counter |
| locked_until | TIMESTAMPTZ | Account lock expiration |
| password_reset_token | VARCHAR(255) | Password reset token |
| password_reset_expires_at | TIMESTAMPTZ | Reset token expiration |
| is_active | BOOLEAN | Account active status |
| created_at | TIMESTAMPTZ | Creation timestamp |
| updated_at | TIMESTAMPTZ | Last update timestamp |
| deleted_at | TIMESTAMPTZ | Soft delete timestamp |

### organizations table
| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| builder_id | UUID | FK to builders |
| name | VARCHAR(255) | Organization name |
| is_default | BOOLEAN | Default org for builder |
| settings | JSONB | Org-specific settings |
| created_at | TIMESTAMPTZ | Creation timestamp |
| updated_at | TIMESTAMPTZ | Last update timestamp |
| deleted_at | TIMESTAMPTZ | Soft delete timestamp |

### organization_members table
| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| builder_id | UUID | FK to builders |
| organization_id | UUID | FK to organizations |
| user_id | UUID | FK to users |
| role | VARCHAR(50) | admin, manager, member, viewer |
| invited_at | TIMESTAMPTZ | Invitation timestamp |
| joined_at | TIMESTAMPTZ | Join timestamp |
| is_active | BOOLEAN | Membership active status |
| created_at | TIMESTAMPTZ | Creation timestamp |
| updated_at | TIMESTAMPTZ | Last update timestamp |

### refresh_tokens table
| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| builder_id | UUID | FK to builders |
| user_id | UUID | FK to users |
| token_hash | VARCHAR(255) | SHA-256 hash of token |
| device_info | VARCHAR(500) | User agent string |
| expires_at | TIMESTAMPTZ | Token expiration |
| created_at | TIMESTAMPTZ | Creation timestamp |
| revoked_at | TIMESTAMPTZ | Revocation timestamp |
| revoked_by | UUID | FK to user who revoked |

### projects table
| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| builder_id | UUID | FK to builders |
| organization_id | UUID | FK to organizations |
| name | VARCHAR(255) | Project name |
| address | VARCHAR(500) | Project address |
| unit_type | VARCHAR(50) | single_family, townhomes, condos, apartments |
| units | INTEGER | Number of units |
| lot_size_acres | DECIMAL(10,2) | Lot size in acres |
| status | VARCHAR(50) | planning, active, on_hold, completed |
| start_date | DATE | Planned start date |
| end_date | DATE | Planned end date |
| description | TEXT | Project description |
| purchase_price | DECIMAL(14,2) | Property purchase price per unit |
| appraised_value | DECIMAL(14,2) | Current appraised value per unit |
| target_arv | DECIMAL(14,2) | After Repair Value per unit (target) |
| valuation_date | DATE | Date of most recent appraisal |
| appraisal_document_id | UUID | FK to documents (appraisal doc) |
| created_at | TIMESTAMPTZ | Creation timestamp |
| updated_at | TIMESTAMPTZ | Last update timestamp |
| deleted_at | TIMESTAMPTZ | Soft delete timestamp |

### activity_log table
| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| builder_id | UUID | FK to builders |
| user_id | UUID | FK to users |
| project_id | UUID | FK to projects (optional) |
| entity_type | VARCHAR(50) | project, task, budget_item, etc. |
| entity_id | UUID | ID of the affected entity |
| action | VARCHAR(50) | created, updated, deleted, status_changed |
| changes | JSONB | Field changes (from/to values) |
| metadata | JSONB | Additional context |
| created_at | TIMESTAMPTZ | Creation timestamp |

### invitations table
| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| builder_id | UUID | FK to builders |
| organization_id | UUID | FK to organizations |
| email | VARCHAR(255) | Invited email address |
| role | VARCHAR(50) | admin, manager, member, viewer |
| token_hash | VARCHAR(255) | SHA-256 hash of invitation token |
| invited_by | UUID | FK to user who sent invite |
| message | TEXT | Optional invitation message |
| expires_at | TIMESTAMPTZ | Token expiration (7 days) |
| accepted_at | TIMESTAMPTZ | When invitation was accepted |
| created_at | TIMESTAMPTZ | Creation timestamp |
| deleted_at | TIMESTAMPTZ | Soft delete timestamp |

## Invited User Access Pattern

When users are invited to an organization, they have a **different `builder_id`** than the organization they're joining. This requires special handling in all controllers:

### The Problem
- User A creates an account → gets `builder_id: AAA`
- User A creates Organization X (under builder AAA)
- User A invites User B to Organization X
- User B registers → gets `builder_id: BBB` (their own tenant)
- User B joins Organization X (which belongs to builder AAA)

If controllers query by `user.builderId`, User B sees nothing because Organization X's data is under builder AAA.

### The Solution

All project-related controllers follow this pattern:

```typescript
// WRONG: Uses user's builder_id (fails for invited users)
const project = await projectModel.findById(projectId, builderId);

// CORRECT: Check org membership regardless of builder_id
const project = await projectModel.findByIdForUser(projectId, userId);
// Then use project.builder_id for subsequent queries
const items = await budgetItemModel.findByProject(project.builder_id, projectId);
```

### Key Functions

| Function | Purpose |
|----------|---------|
| `projectModel.findByIdForUser(projectId, userId)` | Gets project if user is member of its organization |
| `organizationMemberModel.findMembershipByOrgAndUser(orgId, userId)` | Finds membership without requiring builderId |

### Controllers Using This Pattern

All project-scoped controllers use this pattern:
- `project.controller.ts`
- `schedulePhase.controller.ts`
- `scheduleMilestone.controller.ts`
- `scheduleTask.controller.ts`
- `budgetItem.controller.ts`
- `budgetCategory.controller.ts` (accepts `projectId` query param)
- `document.controller.ts`
- `payment.controller.ts`
- `projectContractor.controller.ts`
- `risk.controller.ts`

## Data Flow

### 1. Registration Flow
```
Browser → POST /api/auth/register
                    ↓
            Create Builder (tenant)
                    ↓
            Create User (owner)
                    ↓
            Create Default Organization
                    ↓
            Add User as Admin Member
                    ↓
            Send Verification Email (Resend)
                    ↓
            Return success message
```

### 2. Authentication Flow
```
Browser → POST /api/auth/login
                    ↓
            Validate credentials
                    ↓
            Check email verified
                    ↓
            Generate access token (15 min)
                    ↓
            Generate refresh token (7 days)
                    ↓
            Store refresh token hash in DB
                    ↓
            Set HTTP-only cookies
                    ↓
            Return user data + access token
```

### 3. API Request Flow
```
Browser → Request with access_token cookie
                    ↓
            JWT verification middleware
                    ↓
            Extract user_id and builder_id
                    ↓
            Attach to req.user
                    ↓
            Controller handles request
                    ↓
            All queries filtered by builder_id
```

## Project Templates

When a new project is created, the system automatically populates default schedule and budget templates. These templates can also be manually re-applied via the API.

### Schedule Templates

Located in `src/utils/scheduleTemplates.ts`:

- **12 construction phases**: Pre-Construction, Site Work, Foundation, Framing, Rough-In (MEP), Exterior, Insulation & Drywall, Interior Finishes, Flooring, Fixtures & Appliances, Final, Landscaping
- **50+ tasks** distributed across phases
- **8 milestones** at key project checkpoints
- Dates calculated from project start date using business days (skips weekends)
- Duration multipliers by unit type:
  - Single Family: 1.0x
  - Townhomes: 1.2x
  - Condos: 1.5x
  - Apartments: 1.8x
- Parallel phase scheduling to minimize overall timeline

### Budget Templates

Located in `src/utils/budgetTemplates.ts`:

- **30 budget categories**: Land, Permits, Site Work, Foundation, Framing, Roofing, etc.
- **100+ line items** with per-unit cost estimates
- Amounts scale by unit count
- Cost multipliers by unit type:
  - Single Family: 1.0x (baseline)
  - Townhomes: 0.85x (shared walls/infrastructure)
  - Condos: 0.75x (more shared infrastructure)
  - Apartments: 0.70x (economies of scale)

## Infrastructure

```
┌──────────────────────────────────────────────────────────────────────┐
│                          HarvestIQ System                            │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────┐     ┌─────────────────┐     ┌───────────────┐  │
│  │    Frontend     │────▶│   Backend API   │────▶│   PostgreSQL  │  │
│  │    (Vercel)     │     │   (Railway)     │     │   (Railway)   │  │
│  │   Next.js 15    │     │  Express 5 + TS │     │               │  │
│  └─────────────────┘     └────────┬────────┘     └───────┬───────┘  │
│                                   │                      │          │
│                          ┌────────┼────────┐             │          │
│                          │        │        │             │          │
│                          ▼        ▼        ▼             │          │
│                 ┌──────────┐ ┌────────┐ ┌────────┐       │          │
│                 │Cloudflare│ │ Resend │ │SendGrid│       │          │
│                 │R2 Storage│ │(Email) │ │(Inbound│       │          │
│                 └──────────┘ └────────┘ │ Parse) │       │          │
│                                         └───┬────┘       │          │
│                                             │            │          │
│                                             ▼            │          │
│                                    ┌─────────────────┐   │          │
│                                    │  Worker Service  │───┘          │
│                                    │  (Railway)       │              │
│                                    │  Background Jobs │──▶ Resend    │
│                                    └─────────────────┘   (replies)  │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### Railway Services

| Service | Purpose |
|---------|---------|
| **HarvestIQ-backend** | API server handling all HTTP requests |
| **HarvestIQ-worker** | Background job processor for email processing, PDF conversion, AI classification |
| **Postgres** | PostgreSQL database with attached volume |

### Email Processing Flow

```
External email → SendGrid Inbound Parse → POST /api/webhooks/email/inbound
                                                    ↓
                                          Backend validates sender
                                          Creates email_action record
                                          Queues job for worker
                                                    ↓
                                          Worker picks up job
                                          AI classifies email intent
                                          Parses and executes actions
                                          (add invoice, payment, etc.)
                                                    ↓
                                          Worker sends reply via Resend
                                          (reply-to = project email address)
```

### Email Correction Threading

When a user replies to correct an action (e.g., "change document type to invoice"), the system matches the reply back to the original email using two strategies:

1. **In-Reply-To matching**: Checks `reply_message_id` on the original email action (with bracket normalization)
2. **References header fallback**: Since Amazon SES overrides outbound `Message-ID` headers, `In-Reply-To` often contains an unknown SES ID. The fallback extracts all Message-IDs from the `References` header and matches them against stored `message_id` values on email actions that had replies sent.

The `references_header` column on `email_actions` stores the raw References header for this purpose.

### Jostin AI Email Interface

```
User emails jostinai@parse.harvestiq.thex1.com
  → SendGrid Parse webhook (same endpoint)
  → Webhook detects "jostinai" prefix → branches to handleJostinAiEmail()
  → Looks up sender → gets builderId/userId
  → Finds or creates AI conversation (threaded via In-Reply-To → jostin_email_threads)
  → Runs AI pipeline (classify → extract → query → RAG → generate)
  → Sends formatted HTML reply via Resend
  → Stores outbound Message-ID for threading follow-up replies
```

## Security

### Authentication
- JWT access tokens (15 min expiry)
- HTTP-only refresh tokens (7 day expiry)
- Refresh token rotation on use
- Secure cookie settings in production

### Password Security
- Bcrypt hashing (10 rounds)
- Minimum 8 characters
- Account lockout after 5 failed attempts (15 min)

### Rate Limiting
- Login: 10 requests/minute per IP
- Registration: 5 requests/minute per IP
- Password reset: 3 requests/minute per email
- General API: 100 requests/minute per user

### Headers
- Helmet.js for security headers
- CORS restricted to frontend domain
- No sensitive data in error responses

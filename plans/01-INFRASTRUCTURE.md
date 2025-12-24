# Phase 1: Infrastructure & Authentication

Foundation layer - everything else builds on this.

## Database Setup

- PostgreSQL on Railway (already provisioned)
- Connection pooling configuration
- Migration system setup (using node-pg-migrate or similar)

## User Authentication

- User registration and login
- Password hashing (Argon2id)
- Minimum 8 character passwords
- JWT tokens (15 min access, 7 day refresh)
- Secure HTTP-only cookies for token storage
- Password reset flow with single-use tokens (1 hour expiry)
- Email verification required before full access
- Session invalidation on password change

## Multi-Tenant SaaS Model

### Builders (Tenants)
- Top-level tenant entity
- Represents a paying subscriber (construction company/builder)
- Subscription and billing attached at this level
- All data isolated per builder via builder_id on every table

### Organizations
- Belong to a Builder
- Logical grouping within a builder's account
- Users belong to organizations
- Projects belong to organizations

### Hierarchy
```
Builder (Tenant)
    └── Organizations
            └── Users (via membership)
            └── Projects
                    └── All project data
```

### Invite System
- Builders invite users to their organizations
- Users can belong to multiple organizations within same builder
- Invitation tokens expire after 7 days

## Role-Based Access Control

| Role | Permissions |
|------|-------------|
| Admin | Full access, manage users, all project access, organization settings |
| Member | Access assigned projects, create/edit within assigned projects |

## API Authentication Middleware

- JWT validation on all protected routes
- Builder context extracted from token and injected into request
- Organization context validated against builder
- Role-based permission checking per endpoint
- Request rate limiting using **express-rate-limit** (in-memory store for MVP)
- Subscription status check (block access if cancelled/past_due)

## File Storage (R2)

- Cloudflare R2 bucket configuration
- Upload endpoint through backend API only (no direct R2 access)
- Signed URL generation for downloads (15 min expiry)
- File metadata storage in database
- File type whitelist: PDF, PNG, JPG, JPEG, GIF, WEBP, DOC, DOCX, XLS, XLSX
- File size limit: 50MB per file
- Storage quota per builder (tracked, enforced)

## Background Job System

- Job queue using **pg-boss** (PostgreSQL-based, no additional services needed)
- Workers run in same Express app using pg-boss polling
- Worker processes for:
  - Scheduled notification delivery
  - Email/SMS sending
  - Document text extraction
  - Reminder processing
- Job retry with exponential backoff
- Dead letter queue for failed jobs
- Job status tracking in database

## Notification Infrastructure

- Email service integration (SendGrid)
- SMS service integration (Twilio)
- In-app notification system
- Notification queue for async processing
- Delivery status tracking per channel

---

## Database Tables (Phase 1)

### builders
```sql
CREATE TABLE builders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  subscription_status VARCHAR(50) NOT NULL DEFAULT 'trial',
    -- values: active, trial, cancelled, past_due
  subscription_plan VARCHAR(50) NOT NULL DEFAULT 'free',
    -- values: free, pro, enterprise
  trial_ends_at TIMESTAMP WITH TIME ZONE,
  subscription_started_at TIMESTAMP WITH TIME ZONE,
  subscription_ends_at TIMESTAMP WITH TIME ZONE,
  billing_email VARCHAR(255),
  storage_used_bytes BIGINT NOT NULL DEFAULT 0,
  storage_limit_bytes BIGINT NOT NULL DEFAULT 5368709120, -- 5GB default
  ai_tokens_used_month INTEGER NOT NULL DEFAULT 0,
  ai_tokens_limit_month INTEGER NOT NULL DEFAULT 100000, -- 100k tokens/month default
  ai_tokens_reset_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_builders_subscription_status ON builders(subscription_status);
CREATE INDEX idx_builders_deleted_at ON builders(deleted_at);
```

### organizations
```sql
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  name VARCHAR(255) NOT NULL,
  settings JSONB DEFAULT '{}',
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_organizations_builder_id ON organizations(builder_id);
CREATE INDEX idx_organizations_deleted_at ON organizations(deleted_at);
```

### users
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  email VARCHAR(255) NOT NULL UNIQUE,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  phone VARCHAR(50),
  notification_preferences JSONB DEFAULT '{
    "channels": {"email": true, "sms": false, "in_app": true},
    "types": {
      "reminder": ["email", "in_app"],
      "task_assigned": ["in_app"],
      "task_due": ["email", "in_app"],
      "milestone_approaching": ["email", "in_app"],
      "invoice_due": ["email"],
      "system": ["in_app"]
    }
  }',
  email_verified_at TIMESTAMP WITH TIME ZONE,
  last_login_at TIMESTAMP WITH TIME ZONE,
  is_active BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_users_builder_id ON users(builder_id);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_is_active ON users(is_active);
CREATE INDEX idx_users_deleted_at ON users(deleted_at);
```

### organization_members
```sql
CREATE TABLE organization_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  organization_id UUID NOT NULL REFERENCES organizations(id),
  user_id UUID NOT NULL REFERENCES users(id),
  role VARCHAR(50) NOT NULL DEFAULT 'member',
    -- values: admin, member
  invited_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  joined_at TIMESTAMP WITH TIME ZONE,
  is_active BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  UNIQUE(organization_id, user_id)
);

CREATE INDEX idx_org_members_builder_id ON organization_members(builder_id);
CREATE INDEX idx_org_members_organization_id ON organization_members(organization_id);
CREATE INDEX idx_org_members_user_id ON organization_members(user_id);
CREATE INDEX idx_org_members_role ON organization_members(role);
```

### refresh_tokens
```sql
CREATE TABLE refresh_tokens (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  user_id UUID NOT NULL REFERENCES users(id),
  token_hash VARCHAR(255) NOT NULL,
  device_info VARCHAR(500),
  expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  revoked_at TIMESTAMP WITH TIME ZONE,
  revoked_by UUID REFERENCES users(id)
);

CREATE INDEX idx_refresh_tokens_builder_id ON refresh_tokens(builder_id);
CREATE INDEX idx_refresh_tokens_user_id ON refresh_tokens(user_id);
CREATE INDEX idx_refresh_tokens_token_hash ON refresh_tokens(token_hash);
CREATE INDEX idx_refresh_tokens_expires_at ON refresh_tokens(expires_at);
```

### invitation_tokens
```sql
CREATE TABLE invitation_tokens (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  organization_id UUID NOT NULL REFERENCES organizations(id),
  email VARCHAR(255) NOT NULL,
  role VARCHAR(50) NOT NULL DEFAULT 'member',
  token_hash VARCHAR(255) NOT NULL,
  invited_by UUID NOT NULL REFERENCES users(id),
  expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
  accepted_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_invitation_tokens_builder_id ON invitation_tokens(builder_id);
CREATE INDEX idx_invitation_tokens_token_hash ON invitation_tokens(token_hash);
CREATE INDEX idx_invitation_tokens_email ON invitation_tokens(email);
```

### jobs (Background Job Queue)
```sql
CREATE TABLE jobs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID REFERENCES builders(id),
  queue VARCHAR(100) NOT NULL,
    -- values: notifications, emails, sms, document_processing
  type VARCHAR(100) NOT NULL,
  payload JSONB NOT NULL,
  status VARCHAR(50) NOT NULL DEFAULT 'pending',
    -- values: pending, processing, completed, failed, dead
  priority INTEGER NOT NULL DEFAULT 0,
  attempts INTEGER NOT NULL DEFAULT 0,
  max_attempts INTEGER NOT NULL DEFAULT 3,
  scheduled_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  started_at TIMESTAMP WITH TIME ZONE,
  completed_at TIMESTAMP WITH TIME ZONE,
  failed_at TIMESTAMP WITH TIME ZONE,
  error_message TEXT,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_jobs_queue_status ON jobs(queue, status);
CREATE INDEX idx_jobs_scheduled_at ON jobs(scheduled_at);
CREATE INDEX idx_jobs_builder_id ON jobs(builder_id);
CREATE INDEX idx_jobs_status ON jobs(status);
```

### activity_log
```sql
CREATE TABLE activity_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  organization_id UUID REFERENCES organizations(id),
  project_id UUID,
  user_id UUID REFERENCES users(id),
  action VARCHAR(100) NOT NULL,
    -- values: created, updated, deleted, uploaded, assigned, completed, etc.
  entity_type VARCHAR(100) NOT NULL,
    -- values: project, task, document, payment, invoice, etc.
  entity_id UUID NOT NULL,
  entity_name VARCHAR(255),
  changes JSONB,
  ip_address INET,
  user_agent VARCHAR(500),
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_activity_log_builder_id ON activity_log(builder_id);
CREATE INDEX idx_activity_log_organization_id ON activity_log(organization_id);
CREATE INDEX idx_activity_log_project_id ON activity_log(project_id);
CREATE INDEX idx_activity_log_user_id ON activity_log(user_id);
CREATE INDEX idx_activity_log_entity ON activity_log(entity_type, entity_id);
CREATE INDEX idx_activity_log_created_at ON activity_log(created_at);
```

---

## API Endpoints (Phase 1)

### Authentication
```
POST /auth/register              - Create new builder + admin user
POST /auth/login                 - Login, returns tokens
POST /auth/logout                - Revoke refresh token
POST /auth/refresh               - Refresh access token
POST /auth/forgot-password       - Send password reset email
POST /auth/reset-password        - Reset password with token
POST /auth/verify-email          - Verify email with token
POST /auth/resend-verification   - Resend verification email
```

### Builder (Tenant)
```
GET  /builder                    - Get current builder details
PUT  /builder                    - Update builder settings
GET  /builder/subscription       - Get subscription details
GET  /builder/storage            - Get storage usage
```

### Organizations
```
GET    /organizations                           - List all orgs (paginated)
  ?limit=20&offset=0
POST   /organizations                           - Create organization
GET    /organizations/:id                       - Get organization details
PUT    /organizations/:id                       - Update organization
DELETE /organizations/:id                       - Soft delete organization
```

### Organization Members
```
GET    /organizations/:id/members               - List members (paginated)
  ?role=admin&status=active&limit=20&offset=0
POST   /organizations/:id/members/invite        - Send invitation
POST   /organizations/:id/members/accept        - Accept invitation (with token)
GET    /organizations/:id/members/:userId       - Get member details
PUT    /organizations/:id/members/:userId/role  - Change member role (admin only)
DELETE /organizations/:id/members/:userId       - Remove member (admin only)
```

### Current User
```
GET  /users/me                         - Get current user profile
PUT  /users/me                         - Update profile
PUT  /users/me/notification-preferences - Update notification settings
GET  /users/me/organizations           - List user's organizations
```

### Activity Feed
```
GET /activity                                   - Get activity feed (paginated)
  ?organization_id=X&project_id=X&user_id=X&entity_type=X&limit=50&offset=0
```

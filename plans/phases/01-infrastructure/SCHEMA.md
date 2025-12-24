# Phase 1: Database Schema

Tables created in this phase establish multi-tenant architecture and user management.

---

## Migration Order

Run in sequence (foreign key dependencies):

1. `001_create_builders.sql`
2. `002_create_users.sql`
3. `003_create_organizations.sql`
4. `004_create_organization_members.sql`
5. `005_create_refresh_tokens.sql`

---

## Table: builders

The tenant/company level. All data is isolated by `builder_id`.

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
  settings JSONB DEFAULT '{}',
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_builders_subscription_status ON builders(subscription_status);
CREATE INDEX idx_builders_deleted_at ON builders(deleted_at);
```

### Columns

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| name | VARCHAR(255) | No | - | Company name |
| subscription_status | VARCHAR(50) | No | 'trial' | Billing status |
| subscription_plan | VARCHAR(50) | No | 'free' | Plan level |
| trial_ends_at | TIMESTAMP | Yes | - | Trial expiration |
| subscription_started_at | TIMESTAMP | Yes | - | Subscription start |
| subscription_ends_at | TIMESTAMP | Yes | - | Subscription end |
| billing_email | VARCHAR(255) | Yes | - | Billing contact email |
| storage_used_bytes | BIGINT | No | 0 | Current storage usage |
| storage_limit_bytes | BIGINT | No | 5GB | Storage quota |
| ai_tokens_used_month | INTEGER | No | 0 | Current month AI usage |
| ai_tokens_limit_month | INTEGER | No | 100000 | Monthly AI token limit |
| ai_tokens_reset_at | TIMESTAMP | Yes | - | When tokens reset |
| settings | JSONB | Yes | '{}' | Builder preferences |
| created_at | TIMESTAMP | No | NOW() | Creation timestamp |
| updated_at | TIMESTAMP | No | NOW() | Last update timestamp |
| deleted_at | TIMESTAMP | Yes | - | Soft delete timestamp |

---

## Table: users

Individual user accounts. Each user belongs to exactly one builder.

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  email VARCHAR(255) NOT NULL UNIQUE,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  phone VARCHAR(50),
  timezone VARCHAR(50) DEFAULT 'America/New_York',
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
  email_verification_token VARCHAR(255),
  email_verification_expires_at TIMESTAMP WITH TIME ZONE,
  last_login_at TIMESTAMP WITH TIME ZONE,
  failed_login_attempts INTEGER NOT NULL DEFAULT 0,
  locked_until TIMESTAMP WITH TIME ZONE,
  password_reset_token VARCHAR(255),
  password_reset_expires_at TIMESTAMP WITH TIME ZONE,
  is_active BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_users_builder_id ON users(builder_id);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_is_active ON users(is_active);
CREATE INDEX idx_users_email_verification_token ON users(email_verification_token);
CREATE INDEX idx_users_password_reset_token ON users(password_reset_token);
CREATE INDEX idx_users_deleted_at ON users(deleted_at);
```

### Columns

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| builder_id | UUID | No | - | FK to builders |
| email | VARCHAR(255) | No | - | Unique email |
| password_hash | VARCHAR(255) | No | - | Argon2id hash |
| name | VARCHAR(255) | No | - | Display name |
| phone | VARCHAR(50) | Yes | - | Phone number |
| timezone | VARCHAR(50) | Yes | America/New_York | User timezone |
| notification_preferences | JSONB | Yes | {...} | Notification settings |
| email_verified_at | TIMESTAMP | Yes | - | When email was verified |
| email_verification_token | VARCHAR(255) | Yes | - | Verification token (hashed) |
| email_verification_expires_at | TIMESTAMP | Yes | - | Verification token expiry |
| last_login_at | TIMESTAMP | Yes | - | Last login time |
| failed_login_attempts | INTEGER | No | 0 | Failed login counter |
| locked_until | TIMESTAMP | Yes | - | Account lockout expiry |
| password_reset_token | VARCHAR(255) | Yes | - | Reset token (hashed) |
| password_reset_expires_at | TIMESTAMP | Yes | - | Token expiry |
| is_active | BOOLEAN | No | true | Account status |
| created_at | TIMESTAMP | No | NOW() | Creation timestamp |
| updated_at | TIMESTAMP | No | NOW() | Last update timestamp |
| deleted_at | TIMESTAMP | Yes | - | Soft delete timestamp |

---

## Table: organizations

Business units within a builder. Users can belong to multiple organizations.

```sql
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  name VARCHAR(255) NOT NULL,
  is_default BOOLEAN NOT NULL DEFAULT false,
  settings JSONB DEFAULT '{}',
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_organizations_builder_id ON organizations(builder_id);
CREATE INDEX idx_organizations_deleted_at ON organizations(deleted_at);
```

### Columns

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| builder_id | UUID | No | - | FK to builders |
| name | VARCHAR(255) | No | - | Organization name |
| is_default | BOOLEAN | No | false | Default org for builder |
| settings | JSONB | Yes | '{}' | Organization settings |
| created_at | TIMESTAMP | No | NOW() | Creation timestamp |
| updated_at | TIMESTAMP | No | NOW() | Last update timestamp |
| deleted_at | TIMESTAMP | Yes | - | Soft delete timestamp |

---

## Table: organization_members

Junction table linking users to organizations with roles.

```sql
CREATE TABLE organization_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role VARCHAR(50) NOT NULL DEFAULT 'member',
    -- values: admin, member
  invited_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  joined_at TIMESTAMP WITH TIME ZONE,
  is_active BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  CONSTRAINT uq_org_members_user_org UNIQUE (organization_id, user_id)
);

CREATE INDEX idx_org_members_builder_id ON organization_members(builder_id);
CREATE INDEX idx_org_members_organization_id ON organization_members(organization_id);
CREATE INDEX idx_org_members_user_id ON organization_members(user_id);
CREATE INDEX idx_org_members_role ON organization_members(role);
```

### Columns

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| builder_id | UUID | No | - | FK to builders |
| organization_id | UUID | No | - | FK to organizations |
| user_id | UUID | No | - | FK to users |
| role | VARCHAR(50) | No | 'member' | admin or member |
| invited_at | TIMESTAMP | No | NOW() | When user was invited |
| joined_at | TIMESTAMP | Yes | - | When user accepted |
| is_active | BOOLEAN | No | true | Membership status |

---

## Table: refresh_tokens

Stores hashed refresh tokens for secure token rotation and revocation.

```sql
CREATE TABLE refresh_tokens (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
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

### Columns

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| builder_id | UUID | No | - | FK to builders |
| user_id | UUID | No | - | FK to users |
| token_hash | VARCHAR(255) | No | - | SHA-256 hash of token |
| device_info | VARCHAR(500) | Yes | - | User agent/device info |
| expires_at | TIMESTAMP | No | - | Token expiration |
| created_at | TIMESTAMP | No | NOW() | Creation timestamp |
| revoked_at | TIMESTAMP | Yes | - | When token was revoked |
| revoked_by | UUID | Yes | - | Who revoked (for audit) |

---

## Entity Relationships

```
builders (1) ─────< (many) users
    │                    │
    │                    │
    └────< (many) organizations
                  │
                  │
                  └────< (many) organization_members >──── users

users (1) ────< (many) refresh_tokens
```

---

## Row-Level Security Pattern

All queries must filter by `builder_id` to ensure tenant isolation:

```sql
-- Example: Get user by email (must also verify builder)
SELECT * FROM users
WHERE email = $1
AND builder_id = $2
AND deleted_at IS NULL;

-- Example: Get organizations for a builder
SELECT * FROM organizations
WHERE builder_id = $1
AND deleted_at IS NULL;
```

---

## Indexes Rationale

| Index | Purpose |
|-------|---------|
| `idx_users_email` | Fast email lookup for login |
| `idx_users_builder_id` | Filter users by tenant |
| `idx_users_email_verification_token` | Fast token lookup for verification |
| `idx_users_password_reset_token` | Fast token lookup for reset |
| `idx_organizations_builder_id` | Filter orgs by tenant |
| `idx_org_members_*` | Fast membership lookups |
| `idx_refresh_tokens_token_hash` | Fast token validation |
| `idx_refresh_tokens_user_id` | List/revoke user sessions |
| `idx_*_deleted_at` | Exclude soft-deleted records |

---

## Soft Delete Convention

All tables with `deleted_at` column follow soft delete pattern:

```sql
-- Delete (soft)
UPDATE users SET deleted_at = NOW() WHERE id = $1;

-- All queries exclude deleted records
SELECT * FROM users WHERE deleted_at IS NULL;
```

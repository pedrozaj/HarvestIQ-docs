# Phase 2: Database Schema

Tables for project management and activity tracking.

---

## Migration Order

Run after Phase 1 migrations:

5. `005_create_projects.sql`
6. `006_create_activity_log.sql`

---

## Table: projects

Construction development projects with multi-unit tracking.

```sql
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  organization_id UUID NOT NULL REFERENCES organizations(id),
  name VARCHAR(255) NOT NULL,
  address TEXT,
  unit_type VARCHAR(50) NOT NULL,
    -- values: single_family, townhomes, condos, apartments
  units INTEGER NOT NULL,
    -- Number of units in the development
  lot_size_acres DECIMAL(10,2),
  status VARCHAR(50) NOT NULL DEFAULT 'planning',
    -- values: planning, active, on_hold, completed
  start_date DATE,
  end_date DATE,
  description TEXT,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE,

  -- Ensure organization belongs to same builder
  CONSTRAINT fk_projects_org_same_builder
    CHECK (EXISTS (
      SELECT 1 FROM organizations WHERE id = organization_id AND builder_id = projects.builder_id
    ))
);

CREATE INDEX idx_projects_builder_id ON projects(builder_id);
CREATE INDEX idx_projects_organization_id ON projects(organization_id);
CREATE INDEX idx_projects_status ON projects(status);
CREATE INDEX idx_projects_deleted_at ON projects(deleted_at);
CREATE INDEX idx_projects_created_at ON projects(created_at);
```

### Columns

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| builder_id | UUID | No | - | FK to builders (tenant) |
| organization_id | UUID | No | - | FK to organizations |
| name | VARCHAR(255) | No | - | Project name |
| address | TEXT | Yes | - | Project address |
| unit_type | VARCHAR(50) | No | - | Type of units |
| units | INTEGER | No | - | Number of units |
| lot_size_acres | DECIMAL(10,2) | Yes | - | Total lot size |
| status | VARCHAR(50) | No | 'planning' | Current status |
| start_date | DATE | Yes | - | Project start |
| end_date | DATE | Yes | - | Expected completion |
| description | TEXT | Yes | - | Project notes |
| created_at | TIMESTAMP | No | NOW() | Creation time |
| updated_at | TIMESTAMP | No | NOW() | Last update time |
| deleted_at | TIMESTAMP | Yes | - | Soft delete time |

### Unit Types

| Value | Display | Description |
|-------|---------|-------------|
| single_family | Single Family | Individual houses |
| townhomes | Townhomes | Attached units |
| condos | Condos | Condominium units |
| apartments | Apartments | Rental units |

### Status Values

| Value | Display | Description |
|-------|---------|-------------|
| planning | Planning | Pre-construction |
| active | Active | Under construction |
| on_hold | On Hold | Temporarily paused |
| completed | Completed | Finished |

---

## Table: activity_log

Immutable audit trail for all significant actions.

```sql
CREATE TABLE activity_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  user_id UUID REFERENCES users(id),
  project_id UUID REFERENCES projects(id),
  entity_type VARCHAR(50) NOT NULL,
    -- values: project, task, budget_item, document, payment, invoice, etc.
  entity_id UUID NOT NULL,
  action VARCHAR(50) NOT NULL,
    -- values: created, updated, deleted, status_changed, assigned, completed
  changes JSONB,
    -- { field: { from: oldValue, to: newValue } }
  metadata JSONB,
    -- Additional context
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_activity_log_builder_id ON activity_log(builder_id);
CREATE INDEX idx_activity_log_project_id ON activity_log(project_id);
CREATE INDEX idx_activity_log_user_id ON activity_log(user_id);
CREATE INDEX idx_activity_log_entity ON activity_log(entity_type, entity_id);
CREATE INDEX idx_activity_log_created_at ON activity_log(created_at);
```

### Columns

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| builder_id | UUID | No | - | FK to builders (tenant) |
| user_id | UUID | Yes | - | User who performed action |
| project_id | UUID | Yes | - | Related project (if any) |
| entity_type | VARCHAR(50) | No | - | Type of entity |
| entity_id | UUID | No | - | ID of entity |
| action | VARCHAR(50) | No | - | Action performed |
| changes | JSONB | Yes | - | Field changes |
| metadata | JSONB | Yes | - | Extra context |
| created_at | TIMESTAMP | No | NOW() | When action occurred |

### Action Types

| Value | Description |
|-------|-------------|
| created | Entity was created |
| updated | Entity was updated |
| deleted | Entity was deleted/archived |
| status_changed | Status field changed |
| assigned | User was assigned |
| completed | Task/milestone completed |
| uploaded | Document was uploaded |
| payment_recorded | Payment was recorded |

### Example Changes JSONB

```json
{
  "status": {
    "from": "planning",
    "to": "active"
  },
  "name": {
    "from": "Riverside Phase 1",
    "to": "Riverside Apartments"
  }
}
```

---

## Table: invitations (if not using organization_members for pending)

Optional: Track pending invitations separately.

```sql
CREATE TABLE invitations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  organization_id UUID NOT NULL REFERENCES organizations(id),
  email VARCHAR(255) NOT NULL,
  role VARCHAR(50) NOT NULL DEFAULT 'member',
  token VARCHAR(255) NOT NULL UNIQUE,
  invited_by UUID NOT NULL REFERENCES users(id),
  message TEXT,
  expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
  accepted_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  CONSTRAINT uq_invitations_org_email UNIQUE (organization_id, email)
);

CREATE INDEX idx_invitations_builder_id ON invitations(builder_id);
CREATE INDEX idx_invitations_token ON invitations(token);
CREATE INDEX idx_invitations_email ON invitations(email);
```

---

## Entity Relationships

```
builders (1) ───< (many) projects
    │                │
    │                └───────────────────────────┐
    │                                            │
    └───< (many) activity_log >──────────────────┘
                      │
                      └───< users (who performed action)
```

---

## Activity Log Helper Function

```typescript
// src/models/activity.model.ts
interface LogActivityParams {
  builderId: string;
  userId?: string;
  projectId?: string;
  entityType: string;
  entityId: string;
  action: string;
  changes?: Record<string, { from: unknown; to: unknown }>;
  metadata?: Record<string, unknown>;
}

export async function logActivity(params: LogActivityParams): Promise<void> {
  await db.query(
    `INSERT INTO activity_log
     (builder_id, user_id, project_id, entity_type, entity_id, action, changes, metadata)
     VALUES ($1, $2, $3, $4, $5, $6, $7, $8)`,
    [
      params.builderId,
      params.userId,
      params.projectId,
      params.entityType,
      params.entityId,
      params.action,
      params.changes ? JSON.stringify(params.changes) : null,
      params.metadata ? JSON.stringify(params.metadata) : null,
    ]
  );
}
```

---

## Query Patterns

### Get Projects for Builder

```sql
SELECT * FROM projects
WHERE builder_id = $1
AND deleted_at IS NULL
ORDER BY created_at DESC
LIMIT $2 OFFSET $3;
```

### Get Recent Activity for Project

```sql
SELECT
  al.*,
  u.name as user_name
FROM activity_log al
LEFT JOIN users u ON al.user_id = u.id
WHERE al.project_id = $1
AND al.builder_id = $2
ORDER BY al.created_at DESC
LIMIT 50;
```

### Get Builder-wide Activity

```sql
SELECT
  al.*,
  u.name as user_name,
  p.name as project_name
FROM activity_log al
LEFT JOIN users u ON al.user_id = u.id
LEFT JOIN projects p ON al.project_id = p.id
WHERE al.builder_id = $1
ORDER BY al.created_at DESC
LIMIT 50;
```

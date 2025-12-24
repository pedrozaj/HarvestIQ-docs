# Constants & Enums

Centralized definitions for all constant values used across the application. Import from a single source to ensure consistency.

---

## Backend Constants File

Location: `src/constants/index.ts`

```typescript
// Status Enums
export const PROJECT_STATUS = {
  PLANNING: 'planning',
  ACTIVE: 'active',
  ON_HOLD: 'on_hold',
  COMPLETED: 'completed',
} as const;

export const TASK_STATUS = {
  NOT_STARTED: 'not_started',
  IN_PROGRESS: 'in_progress',
  COMPLETED: 'completed',
  BLOCKED: 'blocked',
} as const;

export const TASK_ITEM_STATUS = {
  PENDING: 'pending',
  IN_PROGRESS: 'in_progress',
  COMPLETED: 'completed',
} as const;

export const MILESTONE_STATUS = {
  UPCOMING: 'upcoming',
  ACHIEVED: 'achieved',
  MISSED: 'missed',
} as const;

export const INVOICE_STATUS = {
  UNPAID: 'unpaid',
  PARTIAL: 'partial',
  PAID: 'paid',
} as const;

export const DOCUMENT_TYPE = {
  INVOICE: 'invoice',
  ESTIMATE: 'estimate',
  CONTRACT: 'contract',
  PERMIT: 'permit',
  PHOTO: 'photo',
  OTHER: 'other',
} as const;

export const UNIT_TYPE = {
  SINGLE_FAMILY: 'single_family',
  TOWNHOMES: 'townhomes',
  CONDOS: 'condos',
  APARTMENTS: 'apartments',
} as const;

export const ORGANIZATION_ROLE = {
  ADMIN: 'admin',
  MEMBER: 'member',
} as const;

export const PRIORITY = {
  LOW: 'low',
  MEDIUM: 'medium',
  HIGH: 'high',
  URGENT: 'urgent',
} as const;

export const PAYMENT_METHOD = {
  CHECK: 'check',
  ACH: 'ach',
  CREDIT_CARD: 'credit_card',
  WIRE: 'wire',
  CASH: 'cash',
  OTHER: 'other',
} as const;

export const NOTIFICATION_TYPE = {
  REMINDER: 'reminder',
  TASK_ASSIGNED: 'task_assigned',
  TASK_DUE: 'task_due',
  MILESTONE_APPROACHING: 'milestone_approaching',
  INVOICE_DUE: 'invoice_due',
  BUDGET_ALERT: 'budget_alert',
  SYSTEM: 'system',
} as const;

export const NOTIFICATION_CHANNEL = {
  EMAIL: 'email',
  SMS: 'sms',
  IN_APP: 'in_app',
} as const;

export const DELIVERY_STATUS = {
  PENDING: 'pending',
  SENT: 'sent',
  FAILED: 'failed',
} as const;

export const AI_QUERY_TYPE = {
  SPENDING: 'spending',
  SCHEDULE: 'schedule',
  BUDGET: 'budget',
  COMPARISON: 'comparison',
  TREND: 'trend',
  DOCUMENT: 'document',
  OPEN_ENDED: 'open_ended',
} as const;

export const AI_MESSAGE_ROLE = {
  USER: 'user',
  ASSISTANT: 'assistant',
} as const;

export const INSIGHT_TYPE = {
  COST_PATTERN: 'cost_pattern',
  SCHEDULE_RISK: 'schedule_risk',
  EFFICIENCY: 'efficiency',
  ANOMALY: 'anomaly',
  RECOMMENDATION: 'recommendation',
} as const;

export const INSIGHT_SEVERITY = {
  INFO: 'info',
  WARNING: 'warning',
  CRITICAL: 'critical',
} as const;

export const ALERT_THRESHOLD_TYPE = {
  PERCENTAGE: 'percentage',
  AMOUNT: 'amount',
} as const;

export const RECURRENCE = {
  NONE: 'none',
  DAILY: 'daily',
  WEEKLY: 'weekly',
  MONTHLY: 'monthly',
} as const;

// Type exports for TypeScript
export type ProjectStatus = typeof PROJECT_STATUS[keyof typeof PROJECT_STATUS];
export type TaskStatus = typeof TASK_STATUS[keyof typeof TASK_STATUS];
export type TaskItemStatus = typeof TASK_ITEM_STATUS[keyof typeof TASK_ITEM_STATUS];
export type MilestoneStatus = typeof MILESTONE_STATUS[keyof typeof MILESTONE_STATUS];
export type InvoiceStatus = typeof INVOICE_STATUS[keyof typeof INVOICE_STATUS];
export type DocumentType = typeof DOCUMENT_TYPE[keyof typeof DOCUMENT_TYPE];
export type UnitType = typeof UNIT_TYPE[keyof typeof UNIT_TYPE];
export type OrganizationRole = typeof ORGANIZATION_ROLE[keyof typeof ORGANIZATION_ROLE];
export type Priority = typeof PRIORITY[keyof typeof PRIORITY];
export type PaymentMethod = typeof PAYMENT_METHOD[keyof typeof PAYMENT_METHOD];
export type NotificationType = typeof NOTIFICATION_TYPE[keyof typeof NOTIFICATION_TYPE];
export type NotificationChannel = typeof NOTIFICATION_CHANNEL[keyof typeof NOTIFICATION_CHANNEL];
export type DeliveryStatus = typeof DELIVERY_STATUS[keyof typeof DELIVERY_STATUS];
export type AiQueryType = typeof AI_QUERY_TYPE[keyof typeof AI_QUERY_TYPE];
export type AiMessageRole = typeof AI_MESSAGE_ROLE[keyof typeof AI_MESSAGE_ROLE];
export type InsightType = typeof INSIGHT_TYPE[keyof typeof INSIGHT_TYPE];
export type InsightSeverity = typeof INSIGHT_SEVERITY[keyof typeof INSIGHT_SEVERITY];
export type AlertThresholdType = typeof ALERT_THRESHOLD_TYPE[keyof typeof ALERT_THRESHOLD_TYPE];
export type Recurrence = typeof RECURRENCE[keyof typeof RECURRENCE];
```

---

## Frontend Constants File

Location: `src/lib/constants.ts`

```typescript
// Re-export backend constants for shared use
export * from './constants-shared';

// Frontend-specific display labels
export const PROJECT_STATUS_LABELS: Record<string, string> = {
  planning: 'Planning',
  active: 'Active',
  on_hold: 'On Hold',
  completed: 'Completed',
};

export const TASK_STATUS_LABELS: Record<string, string> = {
  not_started: 'Not Started',
  in_progress: 'In Progress',
  completed: 'Completed',
  blocked: 'Blocked',
};

export const PRIORITY_LABELS: Record<string, string> = {
  low: 'Low',
  medium: 'Medium',
  high: 'High',
  urgent: 'Urgent',
};

export const UNIT_TYPE_LABELS: Record<string, string> = {
  single_family: 'Single Family',
  townhomes: 'Townhomes',
  condos: 'Condos',
  apartments: 'Apartments',
};

export const DOCUMENT_TYPE_LABELS: Record<string, string> = {
  invoice: 'Invoice',
  estimate: 'Estimate',
  contract: 'Contract',
  permit: 'Permit',
  photo: 'Photo',
  other: 'Other',
};

export const PAYMENT_METHOD_LABELS: Record<string, string> = {
  check: 'Check',
  ach: 'ACH Transfer',
  credit_card: 'Credit Card',
  wire: 'Wire Transfer',
  cash: 'Cash',
  other: 'Other',
};

export const INVOICE_STATUS_LABELS: Record<string, string> = {
  unpaid: 'Unpaid',
  partial: 'Partially Paid',
  paid: 'Paid',
};

// Status colors for UI
export const STATUS_COLORS = {
  project: {
    planning: 'blue',
    active: 'green',
    on_hold: 'yellow',
    completed: 'gray',
  },
  task: {
    not_started: 'gray',
    in_progress: 'blue',
    completed: 'green',
    blocked: 'red',
  },
  priority: {
    low: 'gray',
    medium: 'blue',
    high: 'orange',
    urgent: 'red',
  },
  invoice: {
    unpaid: 'red',
    partial: 'yellow',
    paid: 'green',
  },
  milestone: {
    upcoming: 'blue',
    achieved: 'green',
    missed: 'red',
  },
} as const;
```

---

## API Error Codes

```typescript
export const ERROR_CODES = {
  // Authentication (1xxx)
  AUTH_INVALID_CREDENTIALS: 'AUTH_1001',
  AUTH_TOKEN_EXPIRED: 'AUTH_1002',
  AUTH_TOKEN_INVALID: 'AUTH_1003',
  AUTH_REFRESH_FAILED: 'AUTH_1004',
  AUTH_EMAIL_EXISTS: 'AUTH_1005',
  AUTH_WEAK_PASSWORD: 'AUTH_1006',

  // Authorization (2xxx)
  FORBIDDEN_RESOURCE: 'AUTHZ_2001',
  FORBIDDEN_ACTION: 'AUTHZ_2002',
  ROLE_REQUIRED: 'AUTHZ_2003',

  // Validation (3xxx)
  VALIDATION_FAILED: 'VAL_3001',
  INVALID_FORMAT: 'VAL_3002',
  REQUIRED_FIELD: 'VAL_3003',
  INVALID_ENUM: 'VAL_3004',

  // Resources (4xxx)
  NOT_FOUND: 'RES_4001',
  ALREADY_EXISTS: 'RES_4002',
  CONFLICT: 'RES_4003',

  // Business Logic (5xxx)
  BUDGET_EXCEEDED: 'BIZ_5001',
  CIRCULAR_DEPENDENCY: 'BIZ_5002',
  INVITATION_EXPIRED: 'BIZ_5003',
  AI_TOKEN_LIMIT: 'BIZ_5004',

  // System (9xxx)
  INTERNAL_ERROR: 'SYS_9001',
  SERVICE_UNAVAILABLE: 'SYS_9002',
  RATE_LIMITED: 'SYS_9003',
} as const;
```

---

## Database Enum Creation

Run in migrations to create PostgreSQL enums:

```sql
-- Create all enum types
CREATE TYPE project_status AS ENUM ('planning', 'active', 'on_hold', 'completed');
CREATE TYPE task_status AS ENUM ('not_started', 'in_progress', 'completed', 'blocked');
CREATE TYPE task_item_status AS ENUM ('pending', 'in_progress', 'completed');
CREATE TYPE milestone_status AS ENUM ('upcoming', 'achieved', 'missed');
CREATE TYPE invoice_status AS ENUM ('unpaid', 'partial', 'paid');
CREATE TYPE document_type AS ENUM ('invoice', 'estimate', 'contract', 'permit', 'photo', 'other');
CREATE TYPE unit_type AS ENUM ('single_family', 'townhomes', 'condos', 'apartments');
CREATE TYPE organization_role AS ENUM ('admin', 'member');
CREATE TYPE priority AS ENUM ('low', 'medium', 'high', 'urgent');
CREATE TYPE payment_method AS ENUM ('check', 'ach', 'credit_card', 'wire', 'cash', 'other');
CREATE TYPE notification_type AS ENUM ('reminder', 'task_assigned', 'task_due', 'milestone_approaching', 'invoice_due', 'budget_alert', 'system');
CREATE TYPE notification_channel AS ENUM ('email', 'sms', 'in_app');
CREATE TYPE delivery_status AS ENUM ('pending', 'sent', 'failed');
CREATE TYPE ai_query_type AS ENUM ('spending', 'schedule', 'budget', 'comparison', 'trend', 'document', 'open_ended');
CREATE TYPE ai_message_role AS ENUM ('user', 'assistant');
CREATE TYPE insight_type AS ENUM ('cost_pattern', 'schedule_risk', 'efficiency', 'anomaly', 'recommendation');
CREATE TYPE insight_severity AS ENUM ('info', 'warning', 'critical');
CREATE TYPE alert_threshold_type AS ENUM ('percentage', 'amount');
CREATE TYPE recurrence AS ENUM ('none', 'daily', 'weekly', 'monthly');
```

---

## Usage Guidelines

1. **Never hardcode enum values** - Always import from constants
2. **Database columns** - Use VARCHAR with CHECK constraint or PostgreSQL ENUM
3. **API validation** - Validate against constant arrays
4. **Frontend display** - Use label mappings for user-facing text
5. **Adding new values** - Update in this order: DB migration → Backend constants → Frontend constants

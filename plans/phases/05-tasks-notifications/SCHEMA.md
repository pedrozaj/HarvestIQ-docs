# Phase 5: Database Schema

Task items, reminders, and notification tables.

---

## Prerequisites

Before implementing Phase 5:
- Phases 1-4 must be complete (migrations 001-019)
- Backend constants file: `/src/constants/index.ts`
- Backend types file: `/src/types/index.ts`

---

## Constants to Add (Backend)

Add these to `/src/constants/index.ts`:

```typescript
// Phase 5 Constants
export const TASK_ITEM_STATUS = {
  PENDING: 'pending',
  IN_PROGRESS: 'in_progress',
  COMPLETED: 'completed',
} as const;

export const REMINDER_RECURRENCE = {
  NONE: 'none',
  DAILY: 'daily',
  WEEKLY: 'weekly',
  MONTHLY: 'monthly',
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

export const ALERT_THRESHOLD_TYPE = {
  PERCENTAGE: 'percentage',
  AMOUNT: 'amount',
} as const;

export const REMINDER_ENTITY_TYPE = {
  TASK_ITEM: 'task_item',
  MILESTONE: 'milestone',
  INVOICE: 'invoice',
  CUSTOM: 'custom',
} as const;

export type TaskItemStatus = typeof TASK_ITEM_STATUS[keyof typeof TASK_ITEM_STATUS];
export type ReminderRecurrence = typeof REMINDER_RECURRENCE[keyof typeof REMINDER_RECURRENCE];
export type NotificationType = typeof NOTIFICATION_TYPE[keyof typeof NOTIFICATION_TYPE];
export type NotificationChannel = typeof NOTIFICATION_CHANNEL[keyof typeof NOTIFICATION_CHANNEL];
export type DeliveryStatus = typeof DELIVERY_STATUS[keyof typeof DELIVERY_STATUS];
export type AlertThresholdType = typeof ALERT_THRESHOLD_TYPE[keyof typeof ALERT_THRESHOLD_TYPE];
export type ReminderEntityType = typeof REMINDER_ENTITY_TYPE[keyof typeof REMINDER_ENTITY_TYPE];
```

---

## Types to Add (Backend)

Add these to `/src/types/index.ts`:

```typescript
// Phase 5 Types
export interface TaskItemRow {
  id: string;
  builder_id: string;
  project_id: string | null;
  schedule_task_id: string | null;
  assigned_to: string | null;
  created_by: string;
  title: string;
  description: string | null;
  status: string;
  priority: string;
  due_date: string | null;
  completed_at: string | null;
  completed_by: string | null;
  created_at: string;
  updated_at: string;
  deleted_at: string | null;
}

export interface ReminderRow {
  id: string;
  builder_id: string;
  user_id: string;
  project_id: string | null;
  related_entity_type: string | null;
  related_entity_id: string | null;
  title: string;
  description: string | null;
  remind_at: string;
  recurrence: string;
  is_dismissed: boolean;
  created_at: string;
  updated_at: string;
  deleted_at: string | null;
}

export interface NotificationRow {
  id: string;
  builder_id: string;
  user_id: string;
  type: string;
  title: string;
  message: string;
  data: Record<string, unknown> | null;
  is_read: boolean;
  read_at: string | null;
  created_at: string;
}

export interface NotificationPreferenceRow {
  id: string;
  builder_id: string;
  user_id: string;
  notification_type: string;
  email_enabled: boolean;
  sms_enabled: boolean;
  in_app_enabled: boolean;
  created_at: string;
  updated_at: string;
}

export interface NotificationDeliveryRow {
  id: string;
  builder_id: string;
  notification_id: string;
  channel: string;
  status: string;
  attempts: number;
  max_attempts: number;
  last_attempt_at: string | null;
  sent_at: string | null;
  error_message: string | null;
  created_at: string;
}

export interface AlertThresholdRow {
  id: string;
  builder_id: string;
  project_id: string | null;
  category_id: string | null;
  threshold_type: string;
  threshold_value: number;
  is_active: boolean;
  last_triggered_at: string | null;
  created_at: string;
  updated_at: string;
  deleted_at: string | null;
}
```

---

## Migration Order

```
020_create_task_items.sql
021_create_reminders.sql
022_create_notifications.sql
023_create_notification_preferences.sql
024_create_notification_deliveries.sql
025_create_alert_thresholds.sql
```

---

## Table: task_items

Personal actionable items for team members.

```sql
-- Migration: 020_create_task_items.sql
CREATE TABLE task_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
  schedule_task_id UUID REFERENCES schedule_tasks(id) ON DELETE SET NULL,
  assigned_to UUID REFERENCES users(id) ON DELETE SET NULL,
  created_by UUID NOT NULL REFERENCES users(id),
  title VARCHAR(255) NOT NULL,
  description TEXT,
  status VARCHAR(50) NOT NULL DEFAULT 'pending',
    -- values: pending, in_progress, completed
  priority VARCHAR(20) NOT NULL DEFAULT 'medium',
    -- values: low, medium, high, urgent
  due_date DATE,
  completed_at TIMESTAMP WITH TIME ZONE,
  completed_by UUID REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_task_items_builder_id ON task_items(builder_id);
CREATE INDEX idx_task_items_assigned_to ON task_items(assigned_to) WHERE deleted_at IS NULL;
CREATE INDEX idx_task_items_project_id ON task_items(project_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_task_items_status ON task_items(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_task_items_due_date ON task_items(due_date) WHERE deleted_at IS NULL;
```

---

## Table: reminders

Scheduled reminder notifications.

```sql
-- Migration: 021_create_reminders.sql
CREATE TABLE reminders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
  related_entity_type VARCHAR(50),
    -- values: task_item, milestone, invoice, custom
  related_entity_id UUID,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  remind_at TIMESTAMP WITH TIME ZONE NOT NULL,
  recurrence VARCHAR(20) DEFAULT 'none',
    -- values: none, daily, weekly, monthly
  is_dismissed BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_reminders_builder_id ON reminders(builder_id);
CREATE INDEX idx_reminders_user_id ON reminders(user_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_reminders_remind_at ON reminders(remind_at) WHERE deleted_at IS NULL AND is_dismissed = false;
CREATE INDEX idx_reminders_is_dismissed ON reminders(is_dismissed) WHERE deleted_at IS NULL;
```

---

## Table: notifications

In-app notification records.

```sql
-- Migration: 022_create_notifications.sql
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  type VARCHAR(50) NOT NULL,
    -- values: reminder, task_assigned, task_due, milestone_approaching,
    --         invoice_due, budget_alert, system
  title VARCHAR(255) NOT NULL,
  message TEXT NOT NULL,
  data JSONB,
    -- { entityType, entityId, projectId, ... }
  is_read BOOLEAN NOT NULL DEFAULT false,
  read_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_notifications_builder_id ON notifications(builder_id);
CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_is_read ON notifications(is_read);
CREATE INDEX idx_notifications_created_at ON notifications(created_at DESC);
CREATE INDEX idx_notifications_type ON notifications(type);
```

---

## Table: notification_preferences

User channel preferences per notification type.

```sql
-- Migration: 023_create_notification_preferences.sql
CREATE TABLE notification_preferences (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  notification_type VARCHAR(50) NOT NULL,
  email_enabled BOOLEAN NOT NULL DEFAULT true,
  sms_enabled BOOLEAN NOT NULL DEFAULT false,
  in_app_enabled BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  CONSTRAINT uq_notification_prefs UNIQUE (user_id, notification_type)
);

CREATE INDEX idx_notification_preferences_user_id ON notification_preferences(user_id);
```

---

## Table: notification_deliveries

Track delivery attempts for each channel.

```sql
-- Migration: 024_create_notification_deliveries.sql
CREATE TABLE notification_deliveries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  notification_id UUID NOT NULL REFERENCES notifications(id) ON DELETE CASCADE,
  channel VARCHAR(20) NOT NULL,
    -- values: email, sms, in_app
  status VARCHAR(20) NOT NULL DEFAULT 'pending',
    -- values: pending, sent, failed
  attempts INTEGER NOT NULL DEFAULT 0,
  max_attempts INTEGER NOT NULL DEFAULT 3,
  last_attempt_at TIMESTAMP WITH TIME ZONE,
  sent_at TIMESTAMP WITH TIME ZONE,
  error_message TEXT,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_notification_deliveries_notification_id ON notification_deliveries(notification_id);
CREATE INDEX idx_notification_deliveries_status ON notification_deliveries(status);
CREATE INDEX idx_notification_deliveries_pending ON notification_deliveries(status) WHERE status = 'pending';
```

---

## Table: alert_thresholds

Budget alert configuration.

```sql
-- Migration: 025_create_alert_thresholds.sql
CREATE TABLE alert_thresholds (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
  category_id UUID REFERENCES budget_categories(id) ON DELETE CASCADE,
  threshold_type VARCHAR(20) NOT NULL,
    -- values: percentage, amount
  threshold_value DECIMAL(12,2) NOT NULL,
  is_active BOOLEAN NOT NULL DEFAULT true,
  last_triggered_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_alert_thresholds_builder_id ON alert_thresholds(builder_id);
CREATE INDEX idx_alert_thresholds_project_id ON alert_thresholds(project_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_alert_thresholds_active ON alert_thresholds(is_active) WHERE deleted_at IS NULL;
```

---

## Design Notes

### Notification Preferences
The `notification_preferences` table extends (not replaces) the JSONB `notification_preferences` column in the users table. The users table stores global defaults, while this table stores per-type granular preferences.

### Task Items vs Schedule Tasks
- `schedule_tasks` (Phase 3): Work items attached to schedule phases, for Gantt chart tracking
- `task_items` (Phase 5): Personal actionable items, can optionally link to schedule_tasks

### Soft Deletes
All tables with user-created content include `deleted_at` for consistency with Phases 3-4.

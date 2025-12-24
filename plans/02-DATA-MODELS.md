# Phase 2: Core Data Models

The entities that represent the business domain.

All entities include `builder_id` for tenant isolation. Data is filtered at the builder level first, then organization/project level.

---

## Projects

Central entity representing a multi-unit development (townhomes, single-family homes, condos, apartments).

```sql
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  organization_id UUID NOT NULL REFERENCES organizations(id),
  name VARCHAR(255) NOT NULL,
  address TEXT,
  description TEXT,
  unit_type VARCHAR(50) NOT NULL,
    -- values: single_family, townhomes, condos, apartments
  units INTEGER NOT NULL,
    -- Number of units in the development (e.g., 30 homes, 24 townhomes)
  status VARCHAR(50) NOT NULL DEFAULT 'planning',
    -- values: planning, active, completed, on_hold
  lot_size_acres DECIMAL(10,2),
  start_date DATE,
  target_end_date DATE,
  actual_end_date DATE,
  created_by UUID NOT NULL REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_projects_builder_id ON projects(builder_id);
CREATE INDEX idx_projects_organization_id ON projects(organization_id);
CREATE INDEX idx_projects_status ON projects(status);
CREATE INDEX idx_projects_unit_type ON projects(unit_type);
CREATE INDEX idx_projects_deleted_at ON projects(deleted_at);
```

---

## Schedule

### Phases
```sql
CREATE TABLE schedule_phases (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  start_date DATE,
  end_date DATE,
  status VARCHAR(50) NOT NULL DEFAULT 'not_started',
    -- values: not_started, in_progress, completed
  sort_order INTEGER NOT NULL DEFAULT 0,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_schedule_phases_builder_id ON schedule_phases(builder_id);
CREATE INDEX idx_schedule_phases_project_id ON schedule_phases(project_id);
CREATE INDEX idx_schedule_phases_status ON schedule_phases(status);
CREATE INDEX idx_schedule_phases_sort_order ON schedule_phases(project_id, sort_order);
```

### Milestones
```sql
CREATE TABLE schedule_milestones (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id),
  phase_id UUID REFERENCES schedule_phases(id),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  target_date DATE NOT NULL,
  actual_date DATE,
  status VARCHAR(50) NOT NULL DEFAULT 'pending',
    -- values: pending, achieved, missed
  last_notified_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_schedule_milestones_builder_id ON schedule_milestones(builder_id);
CREATE INDEX idx_schedule_milestones_project_id ON schedule_milestones(project_id);
CREATE INDEX idx_schedule_milestones_phase_id ON schedule_milestones(phase_id);
CREATE INDEX idx_schedule_milestones_status ON schedule_milestones(status);
CREATE INDEX idx_schedule_milestones_target_date ON schedule_milestones(target_date);
```

### Tasks
```sql
CREATE TABLE schedule_tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id),
  phase_id UUID REFERENCES schedule_phases(id),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  planned_start_date DATE,
  planned_end_date DATE,
  actual_start_date DATE,
  actual_end_date DATE,
  status VARCHAR(50) NOT NULL DEFAULT 'not_started',
    -- values: not_started, in_progress, completed, blocked
  priority VARCHAR(20) NOT NULL DEFAULT 'medium',
    -- values: low, medium, high, urgent
  assigned_to UUID REFERENCES users(id),
  last_notified_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE,

  -- Ensure assigned user belongs to same builder
  CONSTRAINT fk_assigned_user_same_builder
    CHECK (assigned_to IS NULL OR EXISTS (
      SELECT 1 FROM users WHERE id = assigned_to AND builder_id = schedule_tasks.builder_id
    ))
);

CREATE INDEX idx_schedule_tasks_builder_id ON schedule_tasks(builder_id);
CREATE INDEX idx_schedule_tasks_project_id ON schedule_tasks(project_id);
CREATE INDEX idx_schedule_tasks_phase_id ON schedule_tasks(phase_id);
CREATE INDEX idx_schedule_tasks_assigned_to ON schedule_tasks(assigned_to);
CREATE INDEX idx_schedule_tasks_status ON schedule_tasks(status);
CREATE INDEX idx_schedule_tasks_planned_end_date ON schedule_tasks(planned_end_date);
```

### Task Dependencies
```sql
CREATE TABLE task_dependencies (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  task_id UUID NOT NULL REFERENCES schedule_tasks(id) ON DELETE CASCADE,
  depends_on_task_id UUID NOT NULL REFERENCES schedule_tasks(id) ON DELETE CASCADE,
  dependency_type VARCHAR(50) NOT NULL DEFAULT 'finish_to_start',
    -- values: finish_to_start, start_to_start, finish_to_finish, start_to_finish
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  -- Prevent self-referencing
  CONSTRAINT no_self_dependency CHECK (task_id != depends_on_task_id),
  -- Unique constraint
  UNIQUE(task_id, depends_on_task_id)
);

CREATE INDEX idx_task_dependencies_builder_id ON task_dependencies(builder_id);
CREATE INDEX idx_task_dependencies_task_id ON task_dependencies(task_id);
CREATE INDEX idx_task_dependencies_depends_on ON task_dependencies(depends_on_task_id);
```

---

## Budgets

### Categories
```sql
CREATE TABLE budget_categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  name VARCHAR(255) NOT NULL,
    -- examples: plumbing, electrical, framing, roofing, permits, labor
  description TEXT,
  is_active BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_budget_categories_builder_id ON budget_categories(builder_id);
CREATE INDEX idx_budget_categories_is_active ON budget_categories(is_active);
CREATE UNIQUE INDEX idx_budget_categories_name ON budget_categories(builder_id, name);
```

### Budget Items
```sql
CREATE TABLE budget_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id),
  category_id UUID NOT NULL REFERENCES budget_categories(id),
  phase_id UUID REFERENCES schedule_phases(id),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  estimated_amount DECIMAL(12,2) NOT NULL DEFAULT 0,
  actual_amount DECIMAL(12,2) NOT NULL DEFAULT 0,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_budget_items_builder_id ON budget_items(builder_id);
CREATE INDEX idx_budget_items_project_id ON budget_items(project_id);
CREATE INDEX idx_budget_items_category_id ON budget_items(category_id);
CREATE INDEX idx_budget_items_phase_id ON budget_items(phase_id);
```

---

## Documents

```sql
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id),
  uploaded_by UUID NOT NULL REFERENCES users(id),
  document_type VARCHAR(50) NOT NULL,
    -- values: invoice, estimate, contract, permit, photo, receipt, other
  name VARCHAR(255) NOT NULL,
  description TEXT,
  file_key VARCHAR(500) NOT NULL,
  file_size BIGINT NOT NULL,
  mime_type VARCHAR(100) NOT NULL,
  extracted_text TEXT,
  text_extracted_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_documents_builder_id ON documents(builder_id);
CREATE INDEX idx_documents_project_id ON documents(project_id);
CREATE INDEX idx_documents_document_type ON documents(document_type);
CREATE INDEX idx_documents_uploaded_by ON documents(uploaded_by);
CREATE INDEX idx_documents_created_at ON documents(created_at);
CREATE INDEX idx_documents_deleted_at ON documents(deleted_at);
```

### Document Tags
```sql
CREATE TABLE document_tags (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  document_id UUID NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  tag VARCHAR(100) NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  UNIQUE(document_id, tag)
);

CREATE INDEX idx_document_tags_builder_id ON document_tags(builder_id);
CREATE INDEX idx_document_tags_document_id ON document_tags(document_id);
CREATE INDEX idx_document_tags_tag ON document_tags(tag);
```

---

## Invoices

```sql
CREATE TABLE invoices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id),
  document_id UUID REFERENCES documents(id),
  category_id UUID REFERENCES budget_categories(id),
  invoice_number VARCHAR(100),
  description TEXT,
  invoice_date DATE,
  due_date DATE,
  total_amount DECIMAL(12,2) NOT NULL,
  paid_amount DECIMAL(12,2) NOT NULL DEFAULT 0,
  status VARCHAR(50) NOT NULL DEFAULT 'unpaid',
    -- values: unpaid, partial, paid
  notes TEXT,
  last_notified_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE,

  -- Ensure paid_amount doesn't exceed total
  CONSTRAINT valid_paid_amount CHECK (paid_amount <= total_amount)
);

CREATE INDEX idx_invoices_builder_id ON invoices(builder_id);
CREATE INDEX idx_invoices_project_id ON invoices(project_id);
CREATE INDEX idx_invoices_document_id ON invoices(document_id);
CREATE INDEX idx_invoices_category_id ON invoices(category_id);
CREATE INDEX idx_invoices_status ON invoices(status);
CREATE INDEX idx_invoices_due_date ON invoices(due_date);
```

---

## Payments

```sql
CREATE TABLE payments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id),
  invoice_id UUID REFERENCES invoices(id),
  budget_item_id UUID REFERENCES budget_items(id),
  category_id UUID REFERENCES budget_categories(id),
  amount DECIMAL(12,2) NOT NULL,
  payment_date DATE NOT NULL,
  payment_method VARCHAR(50),
    -- values: check, credit_card, bank_transfer, cash, other
  reference_number VARCHAR(100),
  description TEXT,
  status VARCHAR(50) NOT NULL DEFAULT 'completed',
    -- values: pending, completed, cancelled
  created_by UUID NOT NULL REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_payments_builder_id ON payments(builder_id);
CREATE INDEX idx_payments_project_id ON payments(project_id);
CREATE INDEX idx_payments_invoice_id ON payments(invoice_id);
CREATE INDEX idx_payments_budget_item_id ON payments(budget_item_id);
CREATE INDEX idx_payments_category_id ON payments(category_id);
CREATE INDEX idx_payments_payment_date ON payments(payment_date);
CREATE INDEX idx_payments_status ON payments(status);
```

---

## Task Items (Actionable Items)

Renamed from "todos" to avoid naming conflicts. These are actionable items/tasks for team members, separate from schedule_tasks which are project schedule items.

```sql
CREATE TABLE task_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  organization_id UUID NOT NULL REFERENCES organizations(id),
  project_id UUID REFERENCES projects(id),
  schedule_task_id UUID REFERENCES schedule_tasks(id),
  created_by UUID NOT NULL REFERENCES users(id),
  assigned_to UUID REFERENCES users(id),
  title VARCHAR(255) NOT NULL,
  description TEXT,
  due_date DATE,
  priority VARCHAR(20) NOT NULL DEFAULT 'medium',
    -- values: low, medium, high, urgent
  status VARCHAR(50) NOT NULL DEFAULT 'pending',
    -- values: pending, in_progress, completed, cancelled
  completed_at TIMESTAMP WITH TIME ZONE,
  completed_by UUID REFERENCES users(id),
  last_notified_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE,

  -- Ensure assigned user belongs to same builder
  CONSTRAINT fk_task_items_assigned_same_builder
    CHECK (assigned_to IS NULL OR EXISTS (
      SELECT 1 FROM users WHERE id = assigned_to AND builder_id = task_items.builder_id
    )),
  -- Ensure created_by user belongs to same builder
  CONSTRAINT fk_task_items_creator_same_builder
    CHECK (EXISTS (
      SELECT 1 FROM users WHERE id = created_by AND builder_id = task_items.builder_id
    )),
  -- Ensure completed_by user belongs to same builder
  CONSTRAINT fk_task_items_completer_same_builder
    CHECK (completed_by IS NULL OR EXISTS (
      SELECT 1 FROM users WHERE id = completed_by AND builder_id = task_items.builder_id
    ))
);

CREATE INDEX idx_task_items_builder_id ON task_items(builder_id);
CREATE INDEX idx_task_items_organization_id ON task_items(organization_id);
CREATE INDEX idx_task_items_project_id ON task_items(project_id);
CREATE INDEX idx_task_items_schedule_task_id ON task_items(schedule_task_id);
CREATE INDEX idx_task_items_assigned_to ON task_items(assigned_to);
CREATE INDEX idx_task_items_status ON task_items(status);
CREATE INDEX idx_task_items_due_date ON task_items(due_date);
CREATE INDEX idx_task_items_priority ON task_items(priority);
```

---

## Reminders

```sql
CREATE TABLE reminders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  user_id UUID NOT NULL REFERENCES users(id),
  organization_id UUID REFERENCES organizations(id),
  project_id UUID REFERENCES projects(id),
  related_type VARCHAR(50),
    -- values: task_item, milestone, invoice, schedule_task, custom
  related_id UUID,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  remind_at TIMESTAMP WITH TIME ZONE NOT NULL,
  recurrence VARCHAR(50) NOT NULL DEFAULT 'none',
    -- values: none, daily, weekly, monthly
  status VARCHAR(50) NOT NULL DEFAULT 'pending',
    -- values: pending, sent, cancelled
  sent_at TIMESTAMP WITH TIME ZONE,
  notification_id UUID,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE,

  -- Ensure user belongs to same builder
  CONSTRAINT fk_reminders_user_same_builder
    CHECK (EXISTS (
      SELECT 1 FROM users WHERE id = user_id AND builder_id = reminders.builder_id
    ))
);

CREATE INDEX idx_reminders_builder_id ON reminders(builder_id);
CREATE INDEX idx_reminders_user_id ON reminders(user_id);
CREATE INDEX idx_reminders_remind_at ON reminders(remind_at);
CREATE INDEX idx_reminders_status ON reminders(status);
CREATE INDEX idx_reminders_related ON reminders(related_type, related_id);
```

---

## Notifications

```sql
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  user_id UUID NOT NULL REFERENCES users(id),
  organization_id UUID REFERENCES organizations(id),
  notification_type VARCHAR(50) NOT NULL,
    -- values: reminder, task_assigned, task_due, milestone_approaching, invoice_due, system
  title VARCHAR(255) NOT NULL,
  body TEXT,
  action_url VARCHAR(500),
  data JSONB DEFAULT '{}',
  priority VARCHAR(20) NOT NULL DEFAULT 'normal',
    -- values: low, normal, high
  read_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE,

  -- Ensure user belongs to same builder
  CONSTRAINT fk_notifications_user_same_builder
    CHECK (EXISTS (
      SELECT 1 FROM users WHERE id = user_id AND builder_id = notifications.builder_id
    ))
);

CREATE INDEX idx_notifications_builder_id ON notifications(builder_id);
CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_notification_type ON notifications(notification_type);
CREATE INDEX idx_notifications_read_at ON notifications(read_at);
CREATE INDEX idx_notifications_created_at ON notifications(created_at);
```

### Notification Deliveries
```sql
CREATE TABLE notification_deliveries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  notification_id UUID NOT NULL REFERENCES notifications(id) ON DELETE CASCADE,
  channel VARCHAR(50) NOT NULL,
    -- values: email, sms, in_app
  status VARCHAR(50) NOT NULL DEFAULT 'pending',
    -- values: pending, sent, failed
  sent_at TIMESTAMP WITH TIME ZONE,
  retry_count INTEGER NOT NULL DEFAULT 0,
  error_message TEXT,
  response_data JSONB,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_notification_deliveries_builder_id ON notification_deliveries(builder_id);
CREATE INDEX idx_notification_deliveries_notification_id ON notification_deliveries(notification_id);
CREATE INDEX idx_notification_deliveries_status ON notification_deliveries(status);
CREATE INDEX idx_notification_deliveries_channel ON notification_deliveries(channel);
```

---

## Alert Thresholds

For configuring budget alerts and other threshold-based notifications.

```sql
CREATE TABLE alert_thresholds (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID REFERENCES projects(id),
  alert_type VARCHAR(50) NOT NULL,
    -- values: budget_percent, budget_amount, schedule_variance_days
  threshold_value DECIMAL(12,2) NOT NULL,
  is_active BOOLEAN NOT NULL DEFAULT true,
  last_triggered_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_alert_thresholds_builder_id ON alert_thresholds(builder_id);
CREATE INDEX idx_alert_thresholds_project_id ON alert_thresholds(project_id);
CREATE INDEX idx_alert_thresholds_alert_type ON alert_thresholds(alert_type);
```

---

## Relationships Diagram

```
Builder (Tenant)
    ├── Organizations
    │       ├── Users (via organization_members)
    │       ├── Projects (multi-unit developments)
    │       │       ├── Schedule Phases
    │       │       │       ├── Schedule Tasks
    │       │       │       │       └── Task Dependencies
    │       │       │       └── Milestones
    │       │       ├── Budget Items
    │       │       │       └── Budget Category
    │       │       ├── Documents
    │       │       │       └── Document Tags
    │       │       ├── Invoices
    │       │       │       └── Payments
    │       │       └── Task Items
    │       └── Task Items (org-level)
    ├── Budget Categories
    ├── Reminders
    ├── Notifications
    │       └── Notification Deliveries
    ├── Alert Thresholds
    └── AI Data (see 04-AI-INTEGRATION.md)
```

---

## API Endpoints (Phase 2)

### Projects
```
GET    /projects                                - List projects (paginated)
  ?organization_id=X&status=active&unit_type=X&limit=20&offset=0&sort=created_at&order=desc
POST   /projects                                - Create project
GET    /projects/:id                            - Get project details
PUT    /projects/:id                            - Update project
DELETE /projects/:id                            - Soft delete project
GET    /projects/:id/summary                    - Get project stats summary
GET    /projects/compare                        - Compare multiple projects
  ?ids=uuid1,uuid2,uuid3
```

### Schedule - Phases
```
GET    /projects/:id/phases                     - List phases (sorted by sort_order)
  ?status=in_progress
POST   /projects/:id/phases                     - Create phase
PUT    /phases/:id                              - Update phase
DELETE /phases/:id                              - Soft delete phase
PUT    /projects/:id/phases/reorder             - Reorder phases
```

### Schedule - Milestones
```
GET    /projects/:id/milestones                 - List milestones
  ?status=pending&phase_id=X&from_date=X&to_date=X
POST   /projects/:id/milestones                 - Create milestone
PUT    /milestones/:id                          - Update milestone
DELETE /milestones/:id                          - Soft delete milestone
GET    /milestones/upcoming                     - Get upcoming milestones across projects
  ?days=30&limit=20
```

### Schedule - Tasks
```
GET    /projects/:id/tasks                      - List tasks
  ?status=blocked&phase_id=X&assigned_to=X&priority=high&limit=50&offset=0
POST   /projects/:id/tasks                      - Create task
GET    /tasks/:id                               - Get task with dependencies
PUT    /tasks/:id                               - Update task
DELETE /tasks/:id                               - Soft delete task
POST   /tasks/:id/dependencies                  - Add dependency
DELETE /tasks/:id/dependencies/:depId           - Remove dependency
GET    /tasks/overdue                           - Get overdue tasks across projects
  ?limit=50
GET    /projects/:id/timeline                   - Get timeline data for Gantt chart
```

### Budget - Categories
```
GET    /budget-categories                       - List categories
  ?is_active=true
POST   /budget-categories                       - Create category
PUT    /budget-categories/:id                   - Update category
DELETE /budget-categories/:id                   - Deactivate category (if not in use)
```

### Budget - Items
```
GET    /projects/:id/budget                     - Get full budget with items
  ?category_id=X&phase_id=X
GET    /projects/:id/budget/summary             - Get budget vs actual summary
POST   /projects/:id/budget-items               - Create budget item
PUT    /budget-items/:id                        - Update budget item
DELETE /budget-items/:id                        - Soft delete budget item
GET    /projects/:id/spending-by-category       - Get spending breakdown by category
GET    /projects/:id/spending-by-phase          - Get spending breakdown by phase
```

### Documents
```
GET    /projects/:id/documents                  - List documents (paginated)
  ?document_type=invoice&tags=X&search=X&limit=20&offset=0&sort=created_at
POST   /projects/:id/documents                  - Upload document
GET    /documents/:id                           - Get document metadata
GET    /documents/:id/download                  - Get signed download URL
PUT    /documents/:id                           - Update document metadata
DELETE /documents/:id                           - Soft delete document
POST   /documents/:id/tags                      - Add tags
DELETE /documents/:id/tags/:tag                 - Remove tag
GET    /documents/search                        - Search documents by content/tags
  ?q=X&project_id=X&document_type=X&limit=20
```

### Invoices
```
GET    /projects/:id/invoices                   - List invoices (paginated)
  ?status=unpaid&category_id=X&from_date=X&to_date=X&limit=20&offset=0
POST   /projects/:id/invoices                   - Create invoice
GET    /invoices/:id                            - Get invoice with payments
PUT    /invoices/:id                            - Update invoice
DELETE /invoices/:id                            - Soft delete invoice
GET    /invoices/:id/payments                   - Get payments for invoice
GET    /invoices/overdue                        - Get overdue invoices across projects
  ?limit=50
```

### Payments
```
GET    /projects/:id/payments                   - List payments (paginated)
  ?category_id=X&from_date=X&to_date=X&status=completed&limit=20&offset=0
POST   /projects/:id/payments                   - Record payment
GET    /payments/:id                            - Get payment details
PUT    /payments/:id                            - Update payment
DELETE /payments/:id                            - Soft delete payment
```

### Task Items
```
GET    /task-items                              - List user's task items (paginated)
  ?assigned_to=me&status=pending&priority=high&project_id=X&due_before=X&limit=20&offset=0
GET    /projects/:id/task-items                 - List project task items
POST   /task-items                              - Create task item
GET    /task-items/:id                          - Get task item details
PUT    /task-items/:id                          - Update task item
DELETE /task-items/:id                          - Soft delete task item
GET    /task-items/overdue                      - Get overdue task items
  ?limit=50
```

### Reminders
```
GET    /reminders                               - List user's reminders (paginated)
  ?status=pending&from_date=X&to_date=X&limit=20&offset=0
POST   /reminders                               - Create reminder
GET    /reminders/:id                           - Get reminder details
PUT    /reminders/:id                           - Update reminder
DELETE /reminders/:id                           - Cancel/delete reminder
```

### Notifications
```
GET    /notifications                           - List notifications (paginated)
  ?read=false&notification_type=X&limit=50&offset=0
GET    /notifications/unread-count              - Get unread count
PATCH  /notifications/:id/read                  - Mark as read
POST   /notifications/read-all                  - Mark all as read
DELETE /notifications/:id                       - Delete notification
```

### Alert Thresholds
```
GET    /alert-thresholds                        - List alert configurations
  ?project_id=X&alert_type=X
POST   /alert-thresholds                        - Create alert threshold
PUT    /alert-thresholds/:id                    - Update threshold
DELETE /alert-thresholds/:id                    - Delete threshold
```

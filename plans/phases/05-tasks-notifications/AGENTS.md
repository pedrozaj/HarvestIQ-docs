# Phase 5: Agent Work Instructions

Detailed instructions for parallel agent execution.

---

## Overview

Phase 5 is split into 3 parallel work streams that can be executed by separate agents:

| Stream | Description | Dependencies | Estimated Items |
|--------|-------------|--------------|-----------------|
| A | Database & Core Backend | Prerequisites | 25 items |
| B | Background Jobs & Email | Stream A1 | 15 items |
| C | Frontend Components | Stream A5 | 22 items |

---

## Agent 1: Prerequisites

**Purpose:** Add constants and types before any other work begins.

**Prompt for Agent:**
```
You are implementing Phase 5 prerequisites for HarvestIQ.

TASK: Add Phase 5 constants and types to the backend.

FILES TO MODIFY:
1. /Users/joey/Projects/X1LLC/HarvestIQ-backend/src/constants/index.ts
2. /Users/joey/Projects/X1LLC/HarvestIQ-backend/src/types/index.ts

REFERENCE:
Read /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/05-tasks-notifications/SCHEMA.md
for the exact constants and types to add (see "Constants to Add" and "Types to Add" sections).

PATTERN TO FOLLOW:
Look at existing constants (DOCUMENT_TYPE, INVOICE_STATUS, etc.) and types (DocumentRow, InvoiceRow, etc.)
in those files for the pattern to follow.

VERIFICATION:
After adding, run: cd /Users/joey/Projects/X1LLC/HarvestIQ-backend && npx tsc --noEmit
to verify no TypeScript errors.

DO NOT create any other files. Only modify the two files listed above.
```

---

## Agent 2: Stream A - Database & Core Backend

**Purpose:** Create all database tables, models, schemas, controllers, and routes.

**Prompt for Agent:**
```
You are implementing Phase 5 Stream A (Database & Core Backend) for HarvestIQ.

TASK: Create database migrations, validation schemas, models, controllers, and routes for:
- Task Items (personal actionable items)
- Reminders (scheduled reminders)
- Notifications (in-app notifications)
- Notification Preferences (user settings per notification type)
- Notification Deliveries (delivery tracking)
- Alert Thresholds (budget alert configuration)

REFERENCE FILES:
- Schema definitions: /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/05-tasks-notifications/SCHEMA.md
- API contracts: /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/05-tasks-notifications/API.md
- Checklist: /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/05-tasks-notifications/CHECKLIST.md (Stream A section)

PATTERN FILES TO FOLLOW:
- Model pattern: /Users/joey/Projects/X1LLC/HarvestIQ-backend/src/models/invoice.model.ts
- Schema pattern: /Users/joey/Projects/X1LLC/HarvestIQ-backend/src/schemas/invoice.schema.ts
- Controller pattern: /Users/joey/Projects/X1LLC/HarvestIQ-backend/src/controllers/invoice.controller.ts
- Routes pattern: /Users/joey/Projects/X1LLC/HarvestIQ-backend/src/routes/invoice.routes.ts

FILES TO CREATE:
Backend (in /Users/joey/Projects/X1LLC/HarvestIQ-backend/):
- migrations/020_create_task_items.sql
- migrations/021_create_reminders.sql
- migrations/022_create_notifications.sql
- migrations/023_create_notification_preferences.sql
- migrations/024_create_notification_deliveries.sql
- migrations/025_create_alert_thresholds.sql
- src/schemas/taskItem.schema.ts
- src/schemas/reminder.schema.ts
- src/schemas/notification.schema.ts
- src/schemas/alertThreshold.schema.ts
- src/models/taskItem.model.ts
- src/models/reminder.model.ts
- src/models/notification.model.ts
- src/models/notificationPreference.model.ts
- src/models/notificationDelivery.model.ts
- src/models/alertThreshold.model.ts
- src/controllers/taskItem.controller.ts
- src/controllers/reminder.controller.ts
- src/controllers/notification.controller.ts
- src/controllers/alertThreshold.controller.ts
- src/routes/taskItem.routes.ts
- src/routes/reminder.routes.ts
- src/routes/notification.routes.ts
- src/routes/alertThreshold.routes.ts

FILES TO MODIFY:
- src/routes/index.ts - Register new routes

IMPORTANT PATTERNS:
- All models must scope by builder_id
- All queries must filter deleted_at IS NULL
- Use activity logging for create/update/delete actions
- Controllers must call requireAuth middleware
- Routes use mergeParams: true for nested routes

VERIFICATION:
1. Run: npm run migrate (should create all 6 tables)
2. Run: npx tsc --noEmit (no TypeScript errors)
3. Start server: npm run dev (should start without errors)
```

---

## Agent 3: Stream B - Background Jobs & Email

**Purpose:** Set up pg-boss job queue and email notification system.

**Prerequisites:** Stream A1 (migrations) must be complete.

**Prompt for Agent:**
```
You are implementing Phase 5 Stream B (Background Jobs & Email) for HarvestIQ.

TASK: Set up pg-boss job queue and create job handlers for notifications and email delivery.

REFERENCE FILES:
- Job definitions: /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/05-tasks-notifications/JOBS.md
- Checklist: /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/05-tasks-notifications/CHECKLIST.md (Stream B section)

EXISTING EMAIL SERVICE:
Check if /Users/joey/Projects/X1LLC/HarvestIQ-backend/src/services/email.service.ts exists.
If it does, use it. If not, create it using Resend (already configured in .env).

FILES TO CREATE:
Backend (in /Users/joey/Projects/X1LLC/HarvestIQ-backend/):
- src/jobs/queue.ts - pg-boss instance and startup
- src/jobs/index.ts - Worker entry point
- src/jobs/handlers/reminder.job.ts
- src/jobs/handlers/dueDates.job.ts
- src/jobs/handlers/budgetAlert.job.ts
- src/jobs/handlers/email.job.ts
- src/jobs/handlers/notification.job.ts
- src/services/notification.service.ts - Helper to create notifications
- src/templates/emails/task-assigned.html
- src/templates/emails/task-due.html
- src/templates/emails/milestone-approaching.html
- src/templates/emails/invoice-due.html
- src/templates/emails/budget-alert.html
- src/templates/emails/reminder.html

FILES TO MODIFY:
- package.json - Add "jobs": "ts-node src/jobs/index.ts" script

DEPENDENCIES:
Run: npm install pg-boss

VERIFICATION:
1. Run: npx tsc --noEmit (no TypeScript errors)
2. Start jobs worker: npm run jobs (should connect to pg-boss)
3. Check that pg-boss tables are created in database
```

---

## Agent 4: Stream C - Frontend Components

**Purpose:** Create all frontend hooks, components, and pages.

**Prerequisites:** Stream A5 (routes) must be complete so API endpoints exist.

**Prompt for Agent:**
```
You are implementing Phase 5 Stream C (Frontend Components) for HarvestIQ.

TASK: Create frontend hooks, components, and pages for task items, notifications, reminders, and settings.

REFERENCE FILES:
- UI wireframes: /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/05-tasks-notifications/UI.md
- API contracts: /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/05-tasks-notifications/API.md
- Checklist: /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/05-tasks-notifications/CHECKLIST.md (Stream C section)

PATTERN FILES TO FOLLOW:
- Hook pattern: /Users/joey/Projects/X1LLC/HarvestIQ/src/hooks/usePayments.ts
- Component pattern: /Users/joey/Projects/X1LLC/HarvestIQ/src/components/payments/
- Modal pattern: /Users/joey/Projects/X1LLC/HarvestIQ/src/components/payments/InvoiceFormModal.tsx
- Tab pattern: /Users/joey/Projects/X1LLC/HarvestIQ/src/components/payments/PaymentsTab.tsx

FILES TO CREATE:
Frontend (in /Users/joey/Projects/X1LLC/HarvestIQ/):
- src/hooks/useTaskItems.ts
- src/hooks/useReminders.ts
- src/hooks/useNotifications.ts
- src/hooks/useNotificationPreferences.ts
- src/hooks/useAlertThresholds.ts
- src/components/tasks/TaskItemsTab.tsx
- src/components/tasks/TaskItemList.tsx
- src/components/tasks/TaskItemRow.tsx
- src/components/tasks/TaskItemFormModal.tsx
- src/components/notifications/NotificationBell.tsx
- src/components/notifications/NotificationDropdown.tsx
- src/components/notifications/NotificationList.tsx
- src/components/notifications/NotificationRow.tsx
- src/components/reminders/ReminderList.tsx
- src/components/reminders/ReminderFormModal.tsx
- src/components/settings/NotificationPreferences.tsx
- src/components/settings/AlertThresholdList.tsx
- src/components/settings/AlertThresholdFormModal.tsx
- src/app/tasks/page.tsx
- src/app/reminders/page.tsx
- src/app/notifications/page.tsx
- src/app/settings/notifications/page.tsx

FILES TO MODIFY:
- src/components/DashboardLayout.tsx - Add NotificationBell to header
- src/app/projects/[id]/page.tsx - Enable Tasks tab (change disabled: true to false)

IMPORTANT PATTERNS:
- Use 'use client' directive for all components
- Follow existing styling with glass, btn-primary, etc.
- Use FormDialog for modals
- Use api helper from @/lib/api for API calls

VERIFICATION:
1. Run: npx tsc --noEmit (no TypeScript errors)
2. Start dev server: npm run dev
3. Navigate to /tasks, /reminders, /notifications pages
4. Verify NotificationBell appears in header
```

---

## Agent 5: Integration Testing

**Purpose:** Verify all features work end-to-end.

**Prerequisites:** Streams A, B, and C must all be complete.

**Prompt for Agent:**
```
You are performing integration testing for Phase 5 of HarvestIQ.

TASK: Test all Phase 5 features and verify they work correctly.

CHECKLIST:
Read /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/05-tasks-notifications/CHECKLIST.md
(see "Integration & Testing" section)

TESTS TO PERFORM:
1. Start both servers:
   - Backend: cd /Users/joey/Projects/X1LLC/HarvestIQ-backend && npm run dev
   - Frontend: cd /Users/joey/Projects/X1LLC/HarvestIQ && npm run dev
   - Jobs: cd /Users/joey/Projects/X1LLC/HarvestIQ-backend && npm run jobs

2. Test Task Items:
   - Navigate to /tasks
   - Create a task item
   - Mark it as complete
   - Delete a task item

3. Test Reminders:
   - Navigate to /reminders
   - Create a reminder
   - Dismiss a reminder

4. Test Notifications:
   - Check NotificationBell in header shows unread count
   - Click bell to see dropdown
   - Navigate to /notifications
   - Mark notification as read
   - Mark all as read

5. Test Settings:
   - Navigate to /settings/notifications
   - Toggle preference switches
   - Save and verify they persist

6. Test Email (if configured):
   - Create a task and assign to a user
   - Check Resend dashboard for email delivery

7. Test Background Jobs:
   - Check pg-boss tables have job records
   - Verify jobs are executing (check logs)

REPORT:
Create a report of all tests performed and their results.
Note any bugs or issues found.
```

---

## Agent 6: Audit

**Purpose:** Final verification that Phase 5 is complete.

**Prerequisites:** Integration testing must be complete.

**Prompt for Agent:**
```
You are auditing Phase 5 implementation for HarvestIQ.

TASK: Verify all Phase 5 deliverables are complete and working.

CHECKLIST:
Read /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/05-tasks-notifications/CHECKLIST.md
(see "Audit Checklist" and "Definition of Done" sections)

VERIFICATION STEPS:
1. Check database:
   - Query: SELECT * FROM _migrations WHERE name LIKE '02%';
   - Verify migrations 020-025 are present

2. Check TypeScript compilation:
   - cd /Users/joey/Projects/X1LLC/HarvestIQ-backend && npx tsc --noEmit
   - cd /Users/joey/Projects/X1LLC/HarvestIQ && npx tsc --noEmit

3. Check API endpoints:
   - GET /task-items
   - GET /reminders
   - GET /notifications
   - GET /alert-thresholds

4. Check frontend pages:
   - /tasks renders without errors
   - /reminders renders without errors
   - /notifications renders without errors
   - /settings/notifications renders without errors

5. Check background jobs:
   - pg-boss tables exist
   - Job worker starts without errors

6. Check activity logging:
   - Create a task item
   - Verify activity log entry exists

REPORT:
Create a Phase 5 audit report with:
- All items verified
- Any issues found
- Recommendations for fixes
```

---

## Execution Summary

| Order | Agent | Stream | Can Start After |
|-------|-------|--------|-----------------|
| 1 | Prerequisites | - | Phase 4 complete |
| 2 | Stream A | Database & Backend | Prerequisites |
| 3 | Stream B | Jobs & Email | Stream A1 (migrations) |
| 3 | Stream C | Frontend | Stream A5 (routes) |
| 4 | Integration | Testing | All streams |
| 5 | Audit | Verification | Integration |

**Note:** Agents 3 (Stream B) and 4 (Stream C) can run in parallel.

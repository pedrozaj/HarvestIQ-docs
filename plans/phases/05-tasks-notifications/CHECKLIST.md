# Phase 5: Implementation Checklist

Organized into parallel work streams for efficient agent execution.

---

## Prerequisites (Must Complete First)

- [x] Phases 1-4 complete (migrations 001-019)
- [ ] Add Phase 5 constants to `/src/constants/index.ts` (see SCHEMA.md)
- [ ] Add Phase 5 types to `/src/types/index.ts` (see SCHEMA.md)

---

## Work Stream A: Database & Core Backend

**Agent Context:** This stream creates the database tables and core CRUD models. Must complete before Streams B and C can use the models.

**Files to reference:**
- Schema definitions: `SCHEMA.md`
- Existing model pattern: `/src/models/budgetItem.model.ts`
- Existing schema pattern: `/src/schemas/invoice.schema.ts`

### A1: Database Migrations
- [ ] Create `migrations/020_create_task_items.sql`
- [ ] Create `migrations/021_create_reminders.sql`
- [ ] Create `migrations/022_create_notifications.sql`
- [ ] Create `migrations/023_create_notification_preferences.sql`
- [ ] Create `migrations/024_create_notification_deliveries.sql`
- [ ] Create `migrations/025_create_alert_thresholds.sql`
- [ ] Run `npm run migrate` and verify all tables created

### A2: Validation Schemas
- [ ] Create `src/schemas/taskItem.schema.ts`
- [ ] Create `src/schemas/reminder.schema.ts`
- [ ] Create `src/schemas/notification.schema.ts`
- [ ] Create `src/schemas/alertThreshold.schema.ts`

### A3: Backend Models
- [ ] Create `src/models/taskItem.model.ts` - CRUD + complete + findByUser
- [ ] Create `src/models/reminder.model.ts` - CRUD + dismiss + findDue
- [ ] Create `src/models/notification.model.ts` - CRUD + markRead + markAllRead + getUnreadCount
- [ ] Create `src/models/notificationPreference.model.ts` - get/upsert per user+type
- [ ] Create `src/models/notificationDelivery.model.ts` - create + updateStatus
- [ ] Create `src/models/alertThreshold.model.ts` - CRUD + findActive + markTriggered

### A4: Backend Controllers
- [ ] Create `src/controllers/taskItem.controller.ts`
- [ ] Create `src/controllers/reminder.controller.ts`
- [ ] Create `src/controllers/notification.controller.ts`
- [ ] Create `src/controllers/alertThreshold.controller.ts`

### A5: Backend Routes
- [ ] Create `src/routes/taskItem.routes.ts`
- [ ] Create `src/routes/reminder.routes.ts`
- [ ] Create `src/routes/notification.routes.ts`
- [ ] Create `src/routes/alertThreshold.routes.ts`
- [ ] Add notification preference routes to user routes
- [ ] Register all routes in `src/routes/index.ts`

---

## Work Stream B: Background Jobs & Email

**Agent Context:** This stream sets up the job queue and email templates. Can start after A1 (migrations) is complete. Requires pg-boss package.

**Files to reference:**
- Job definitions: `JOBS.md`
- Email service: Resend is already configured (see `/src/services/email.service.ts` if exists)

### B1: pg-boss Setup
- [ ] Install pg-boss: `npm install pg-boss`
- [ ] Create `src/jobs/queue.ts` - Boss instance and startup
- [ ] Create `src/jobs/index.ts` - Worker entry point
- [ ] Add job worker to `package.json` scripts: `"jobs": "ts-node src/jobs/index.ts"`

### B2: Job Handlers
- [ ] Create `src/jobs/handlers/reminder.job.ts` - Check due reminders
- [ ] Create `src/jobs/handlers/dueDates.job.ts` - Daily due date check
- [ ] Create `src/jobs/handlers/budgetAlert.job.ts` - Threshold check
- [ ] Create `src/jobs/handlers/email.job.ts` - Email delivery worker
- [ ] Create `src/jobs/handlers/notification.job.ts` - Notification creation

### B3: Email Templates
- [ ] Create `src/templates/emails/task-assigned.html`
- [ ] Create `src/templates/emails/task-due.html`
- [ ] Create `src/templates/emails/milestone-approaching.html`
- [ ] Create `src/templates/emails/invoice-due.html`
- [ ] Create `src/templates/emails/budget-alert.html`
- [ ] Create `src/templates/emails/reminder.html`

### B4: Notification Service
- [ ] Create `src/services/notification.service.ts` - Helper to create notifications with preference checks

---

## Work Stream C: Frontend Components

**Agent Context:** This stream creates all frontend UI. Can start after A5 (routes) is complete so API endpoints exist.

**Files to reference:**
- UI wireframes: `UI.md`
- API contracts: `API.md`
- Existing hook pattern: `/src/hooks/usePayments.ts`
- Existing component pattern: `/src/components/payments/`

### C1: Frontend Hooks
- [ ] Create `src/hooks/useTaskItems.ts` - CRUD + complete + filters
- [ ] Create `src/hooks/useReminders.ts` - CRUD + dismiss
- [ ] Create `src/hooks/useNotifications.ts` - list + markRead + markAllRead + unreadCount
- [ ] Create `src/hooks/useNotificationPreferences.ts` - get/update
- [ ] Create `src/hooks/useAlertThresholds.ts` - CRUD

### C2: Task Items Components
- [ ] Create `src/components/tasks/TaskItemsTab.tsx` - Main container
- [ ] Create `src/components/tasks/TaskItemList.tsx` - Grouped list (Overdue/Today/Upcoming)
- [ ] Create `src/components/tasks/TaskItemRow.tsx` - Individual task with checkbox
- [ ] Create `src/components/tasks/TaskItemFormModal.tsx` - Create/edit modal

### C3: Notification Components
- [ ] Create `src/components/notifications/NotificationBell.tsx` - Header bell with badge
- [ ] Create `src/components/notifications/NotificationDropdown.tsx` - Quick view dropdown
- [ ] Create `src/components/notifications/NotificationList.tsx` - Full page list
- [ ] Create `src/components/notifications/NotificationRow.tsx` - Individual notification

### C4: Reminder Components
- [ ] Create `src/components/reminders/ReminderList.tsx` - Reminder display
- [ ] Create `src/components/reminders/ReminderFormModal.tsx` - Create/edit modal

### C5: Settings Components
- [ ] Create `src/components/settings/NotificationPreferences.tsx` - Preference toggles
- [ ] Create `src/components/settings/AlertThresholdList.tsx` - Threshold management
- [ ] Create `src/components/settings/AlertThresholdFormModal.tsx` - Create/edit modal

### C6: Pages & Navigation
- [ ] Create `src/app/tasks/page.tsx` - Task items page
- [ ] Create `src/app/reminders/page.tsx` - Reminders page
- [ ] Create `src/app/notifications/page.tsx` - Full notification history
- [ ] Create `src/app/settings/notifications/page.tsx` - Notification settings
- [ ] Add NotificationBell to header/DashboardLayout
- [ ] Enable Tasks tab in project detail page
- [ ] Add Tasks and Reminders to sidebar navigation

---

## Integration & Testing

**Agent Context:** Run after all streams complete. Verify end-to-end functionality.

### Testing Checklist
- [ ] Test task item CRUD and completion flow
- [ ] Test reminder creation and dismissal
- [ ] Test notification creation via API
- [ ] Test notification bell badge updates
- [ ] Test mark as read / mark all read
- [ ] Test notification preference toggles
- [ ] Test email delivery (check Resend dashboard)
- [ ] Test budget alert threshold triggers
- [ ] Test pg-boss job execution
- [ ] Verify activity logging for all actions

---

## Audit Checklist

**Run after all implementation is complete:**

- [ ] All migrations ran successfully (check `_migrations` table)
- [ ] All API endpoints return expected responses
- [ ] Frontend compiles without TypeScript errors (`npx tsc --noEmit`)
- [ ] Backend compiles without TypeScript errors (`npx tsc --noEmit`)
- [ ] All CRUD operations work in UI
- [ ] Notifications appear in bell dropdown
- [ ] Email notifications are sent (check Resend logs)
- [ ] Background jobs are running (check pg-boss tables)
- [ ] Activity log shows Phase 5 events
- [ ] No console errors in browser

---

## Definition of Done

- [ ] Task items can be created, assigned, and completed
- [ ] Reminders trigger notifications at scheduled time
- [ ] Notifications appear in bell dropdown with unread count
- [ ] Notification page shows full history with filters
- [ ] Email notifications send correctly via Resend
- [ ] User preferences control notification channels
- [ ] Budget alerts trigger when thresholds exceeded
- [ ] Background jobs run reliably via pg-boss
- [ ] All TypeScript compiles without errors
- [ ] Manual testing confirms all features work

---

## Agent Execution Order

```
1. Prerequisites (constants & types)
   |
   v
2. Stream A: Database & Core Backend
   |
   +---> 3. Stream B: Background Jobs (after A1)
   |
   +---> 4. Stream C: Frontend (after A5)
   |
   v
5. Integration & Testing (after A, B, C complete)
   |
   v
6. Audit (final verification)
```

**Parallel execution possible:**
- Stream B can start after Stream A1 (migrations) completes
- Stream C can start after Stream A5 (routes) completes
- Streams B and C can run in parallel

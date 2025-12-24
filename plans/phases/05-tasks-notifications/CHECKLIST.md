# Phase 5: Implementation Checklist

---

## Database

- [ ] Create migrations for all 6 tables
- [ ] Run migrations and verify
- [ ] Seed default notification preferences

---

## pg-boss Setup

- [ ] Install pg-boss
- [ ] Create `src/jobs/queue.ts` with boss setup
- [ ] Configure job schedules
- [ ] Create worker process entry point

---

## Backend Models

- [ ] `taskItem.model.ts` - CRUD + complete
- [ ] `reminder.model.ts` - CRUD + dismiss
- [ ] `notification.model.ts` - CRUD + read
- [ ] `notificationPreference.model.ts` - get/update
- [ ] `alertThreshold.model.ts` - CRUD

---

## Backend Controllers

- [ ] `taskItem.controller.ts`
- [ ] `reminder.controller.ts`
- [ ] `notification.controller.ts`
- [ ] `alertThreshold.controller.ts`

---

## Backend Routes

- [ ] `/task-items` routes
- [ ] `/reminders` routes
- [ ] `/notifications` routes
- [ ] `/users/me/notification-preferences` routes
- [ ] `/alert-thresholds` routes

---

## Background Jobs

- [ ] `reminder.job.ts` - Check due reminders
- [ ] `due-dates.job.ts` - Daily due date check
- [ ] `budget-alert.job.ts` - Threshold check
- [ ] `email.job.ts` - Email delivery
- [ ] `notification.job.ts` - Notification creation

---

## Email Service

- [ ] Configure Resend/SendGrid
- [ ] Create email templates:
  - [ ] task-assigned
  - [ ] task-due
  - [ ] milestone-approaching
  - [ ] invoice-due
  - [ ] budget-alert
  - [ ] reminder

---

## Frontend Components

- [ ] `TaskItemList.tsx`
- [ ] `TaskItemRow.tsx`
- [ ] `TaskItemForm.tsx`
- [ ] `NotificationBell.tsx`
- [ ] `NotificationList.tsx`
- [ ] `ReminderList.tsx`
- [ ] `ReminderForm.tsx`
- [ ] `NotificationPreferences.tsx`

---

## Frontend Pages

- [ ] `/task-items/page.tsx`
- [ ] `/reminders/page.tsx`
- [ ] `/notifications/page.tsx`
- [ ] `/settings/notifications/page.tsx`

---

## Frontend Hooks

- [ ] `useTaskItems(filters)`
- [ ] `useReminders()`
- [ ] `useNotifications()`
- [ ] `useUnreadCount()` - For bell badge
- [ ] `useNotificationPreferences()`

---

## Real-time Updates (Optional)

- [ ] WebSocket or polling for new notifications
- [ ] Update bell badge on new notification

---

## Activity Logging

- [ ] Log task item created/completed/deleted
- [ ] Log reminder created/dismissed
- [ ] Log alert threshold created/triggered

---

## Testing

- [ ] Test task item CRUD and completion
- [ ] Test reminder scheduling
- [ ] Test notification creation
- [ ] Test email delivery
- [ ] Test preference respect
- [ ] Test budget alerts trigger correctly
- [ ] Test pg-boss job execution

---

## Definition of Done

- [ ] Task items can be created and completed
- [ ] Reminders trigger notifications at scheduled time
- [ ] Notifications appear in bell dropdown
- [ ] Notification page shows full history
- [ ] Email notifications send correctly
- [ ] Preferences control notification channels
- [ ] Budget alerts trigger on threshold
- [ ] Background jobs run reliably

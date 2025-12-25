# Phase 5: Task Items & Notifications

Personal productivity and communication features.

---

## Phase Goal

Enable personal task item management, reminders, and a notification system with email delivery.

---

## Prerequisites

| Requirement | Status |
|-------------|--------|
| Phase 1-2: Core Models | Complete |
| Phase 3: Schedule & Budget | Complete |
| Phase 4: Documents & Payments | Complete |
| Migrations 001-019 | Complete |

---

## Deliverables

1. Personal task items with completion tracking
2. Reminder system with scheduled notifications
3. In-app notification bell and history
4. Email notifications for due dates and assignments
5. User notification preferences
6. Budget alert thresholds

---

## Tables Introduced

| Table | Purpose | Migration |
|-------|---------|-----------|
| `task_items` | Personal actionable items | 020 |
| `reminders` | Scheduled reminders | 021 |
| `notifications` | In-app notifications | 022 |
| `notification_preferences` | User channel preferences | 023 |
| `notification_deliveries` | Delivery tracking | 024 |
| `alert_thresholds` | Budget alert configuration | 025 |

---

## Work Streams

Phase 5 is organized into parallel work streams for efficient agent execution:

| Stream | Description | Dependencies |
|--------|-------------|--------------|
| **A** | Database & Core Backend | Prerequisites |
| **B** | Background Jobs & Email | Stream A1 |
| **C** | Frontend Components | Stream A5 |

```
Prerequisites
     |
     v
Stream A (Backend)
     |
     +---> Stream B (Jobs) [parallel]
     |
     +---> Stream C (Frontend) [parallel]
     |
     v
Integration & Testing
     |
     v
Audit
```

See [AGENTS.md](./AGENTS.md) for detailed agent work instructions.

---

## Endpoints Introduced

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET/POST | `/task-items` | Task item management |
| PUT | `/task-items/:id/complete` | Mark complete |
| GET/POST | `/reminders` | Reminder management |
| PUT | `/reminders/:id/dismiss` | Dismiss reminder |
| GET | `/notifications` | Notification list |
| PATCH | `/notifications/:id/read` | Mark read |
| POST | `/notifications/read-all` | Mark all read |
| GET/PUT | `/users/me/notification-preferences` | Preferences |
| GET/POST | `/alert-thresholds` | Alert config |

---

## UI Pages Introduced

| Route | Page | Purpose |
|-------|------|---------|
| /tasks | Task Items | Personal task list |
| /reminders | Reminders | Reminder management |
| /notifications | Notifications | Full history |
| /settings/notifications | Preferences | Notification settings |

---

## Background Jobs

| Job | Schedule | Purpose |
|-----|----------|---------|
| reminder-check | Every minute | Check for due reminders |
| due-date-notifications | Daily 8 AM | Due date reminders |
| budget-alert-check | On payment | Check thresholds |
| send-email | Queue | Email delivery |
| send-notification | Queue | Notification creation |

---

## Related Documentation

| Document | Purpose |
|----------|---------|
| [SCHEMA.md](./SCHEMA.md) | Database tables & constants |
| [API.md](./API.md) | API endpoint contracts |
| [UI.md](./UI.md) | UI wireframes |
| [JOBS.md](./JOBS.md) | Background job definitions |
| [CHECKLIST.md](./CHECKLIST.md) | Implementation checklist |
| [AGENTS.md](./AGENTS.md) | Agent work instructions |

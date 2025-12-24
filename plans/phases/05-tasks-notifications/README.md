# Phase 5: Task Items & Notifications

Personal productivity and communication features.

---

## Phase Goal

Enable personal task item management, reminders, and a notification system with email delivery.

---

## Dependencies

- Phase 2: Core Models (complete)
- Can run parallel to Phases 3 & 4

---

## Deliverables

1. Personal task items with completion tracking
2. Reminder system with scheduled notifications
3. In-app notification bell and history
4. Email notifications for due dates and assignments
5. User notification preferences

---

## Tables Introduced

| Table | Purpose |
|-------|---------|
| `task_items` | Personal actionable items |
| `reminders` | Scheduled reminders |
| `notifications` | In-app notifications |
| `notification_preferences` | User channel preferences |
| `notification_deliveries` | Delivery tracking |
| `alert_thresholds` | Budget alert configuration |

---

## Endpoints Introduced

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET/POST | `/task-items` | Task item management |
| PUT | `/task-items/:id/complete` | Mark complete |
| GET/POST | `/reminders` | Reminder management |
| GET | `/notifications` | Notification list |
| PATCH | `/notifications/:id/read` | Mark read |
| POST | `/notifications/read-all` | Mark all read |
| GET/PUT | `/users/me/notification-preferences` | Preferences |
| GET/POST | `/alert-thresholds` | Alert config |

---

## UI Pages Introduced

| Route | Page | Purpose |
|-------|------|---------|
| /task-items | Task Items | Personal task list |
| /reminders | Reminders | Reminder management |
| /notifications | Notifications | Full history |
| /settings/notifications | Preferences | Notification settings |

---

## Background Jobs

- [ ] Reminder check (every minute)
- [ ] Due date notifications (daily)
- [ ] Budget alert check (on payment)
- [ ] Email delivery worker

---

## Related Documentation

- [SCHEMA.md](./SCHEMA.md) | [API.md](./API.md) | [UI.md](./UI.md) | [JOBS.md](./JOBS.md) | [CHECKLIST.md](./CHECKLIST.md)

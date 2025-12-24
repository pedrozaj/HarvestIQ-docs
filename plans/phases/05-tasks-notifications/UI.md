# Phase 5: UI Pages

Task items, reminders, and notification interfaces.

---

## Task Items Page

Route: `/task-items`

```
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  MY TASK ITEMS                                       [+ New Task]      |
| Projects |                                                                        |
| Tasks    |  [All v] [All Priorities v] [All Projects v] [_______Search______]     |
|          |                                                                        |
| ──────── |  OVERDUE (2)                                                           |
| Team     |  +---------------------------------------------------------------+    |
| Settings |  | [ ] Review permit application  | Riverside | High | Dec 18   |    |
|          |  | [ ] Call electrician           | Oak Grove | Med  | Dec 20   |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  TODAY                                                                 |
|          |  +---------------------------------------------------------------+    |
|          |  | [ ] Submit foundation report   | Riverside | High | Dec 23   |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  UPCOMING                                                              |
|          |  +---------------------------------------------------------------+    |
|          |  | [ ] Schedule framing inspection| Summit    | Med  | Dec 28   |    |
|          |  | [ ] Order cabinets             | Riverside | Low  | Jan 5    |    |
|          |  +---------------------------------------------------------------+    |
+-----------------------------------------------------------------------------------+
```

### Task Item Row

- Checkbox to complete
- Title (click to edit)
- Project name
- Priority badge
- Due date (red if overdue)
- Actions menu (Edit, Delete)

---

## Notification Bell (Header)

```
                                              +---------------------------+
                                              | NOTIFICATIONS             |
                                              +---------------------------+
                                              |                           |
[Bell(3)]  ---------------------------------> | Task assigned to you      |
                                              | Joey assigned "Review..." |
                                              | 5 minutes ago        [•]  |
                                              +---------------------------+
                                              | Milestone approaching     |
                                              | Framing Complete in 2 days|
                                              | 1 hour ago                |
                                              +---------------------------+
                                              | Invoice overdue           |
                                              | INV-001 is 3 days overdue |
                                              | 2 hours ago               |
                                              +---------------------------+
                                              |                           |
                                              | [View All Notifications]  |
                                              +---------------------------+
```

---

## Notifications Page

Route: `/notifications`

```
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  NOTIFICATIONS                              [Mark All Read]            |
| Projects |                                                                        |
| Tasks    |  [All Types v] [Unread Only]                                           |
|          |                                                                        |
| ──────── |  TODAY                                                                 |
| Team     |  +---------------------------------------------------------------+    |
| Settings |  | [•] Task assigned: Review permit application                   |    |
|          |  |     Joey assigned this to you                    5 min ago    |    |
|          |  +---------------------------------------------------------------+    |
|          |  |     Milestone approaching: Framing Complete                    |    |
|          |  |     Due in 2 days on Riverside Apts             1 hour ago    |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  YESTERDAY                                                             |
|          |  +---------------------------------------------------------------+    |
|          |  |     Budget alert: Plumbing 90% of budget                       |    |
|          |  |     Riverside Apartments                        Yesterday     |    |
|          |  +---------------------------------------------------------------+    |
+-----------------------------------------------------------------------------------+
```

---

## Reminders Page

Route: `/reminders`

```
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  REMINDERS                                       [+ New Reminder]      |
| Projects |                                                                        |
| Tasks    |  UPCOMING                                                              |
|          |  +---------------------------------------------------------------+    |
| ──────── |  | Call permit office               | Dec 24, 9:00 AM | [Dismiss]|    |
| Team     |  | Follow up on cabinet order       | Dec 26, 2:00 PM | [Dismiss]|    |
| Settings |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  RECURRING                                                             |
|          |  +---------------------------------------------------------------+    |
|          |  | Weekly project review            | Every Monday 9AM | [Edit]  |    |
|          |  +---------------------------------------------------------------+    |
+-----------------------------------------------------------------------------------+
```

---

## Settings - Notifications

Route: `/settings/notifications`

```
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  SETTINGS                                                              |
| Projects |                                                                        |
| Tasks    |  [Profile] [Notifications] [Security]                                  |
|          |  ===============================                                       |
| ──────── |                                                                        |
| Team     |  NOTIFICATION PREFERENCES                                              |
| Settings |                                                                        |
|          |  +---------------------------------------------------------------+    |
|          |  | Type                  | Email | In-App |                       |    |
|          |  +---------------------------------------------------------------+    |
|          |  | Task Assigned         | [x]   | [x]    |                       |    |
|          |  | Task Due              | [x]   | [x]    |                       |    |
|          |  | Milestone Approaching | [x]   | [x]    |                       |    |
|          |  | Invoice Due           | [x]   | [ ]    |                       |    |
|          |  | Budget Alerts         | [x]   | [x]    |                       |    |
|          |  | Reminders             | [x]   | [x]    |                       |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |                                              [Save Preferences]        |
+-----------------------------------------------------------------------------------+
```

---

## Components to Build

| Component | Purpose |
|-----------|---------|
| TaskItemList | Grouped task display |
| TaskItemRow | Individual task |
| TaskItemForm | Create/edit modal |
| NotificationBell | Header dropdown |
| NotificationList | Full notification page |
| NotificationRow | Individual notification |
| ReminderList | Reminder display |
| ReminderForm | Create/edit modal |
| NotificationPreferences | Settings form |

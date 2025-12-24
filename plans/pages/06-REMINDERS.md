# Reminders

Manage personal reminders for the current user.

Route: `/reminders`

---

## Full Page Layout

```
+-----------------------------------------------------------------------------------+
| [Logo] HarvestIQ     [_____Search_____]              [AI] [Bell(3)] [Avatar]      |
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  MY REMINDERS                                       [+ Add Reminder]   |
| Projects |                                                                        |
| Tasks    |  +---------------------------------------------------------------+    |
|          |  | Status: [Pending v]   Date Range: [This Week v]               |    |
|          |  +---------------------------------------------------------------+    |
| ──────── |                                                                        |
| Team     |  TODAY                                                                 |
| Settings |  +---------------------------------------------------------------+    |
|          |  | 9:00 AM  | Call permit office about inspection        | [...]  |   |
|          |  |          | Related: Permit Approval (Riverside)               |   |
|          |  +---------------------------------------------------------------+    |
|          |  | 2:00 PM  | Follow up with electrician on quote         | [...]  |   |
|          |  |          | Custom reminder                                    |   |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  TOMORROW                                                              |
|          |  +---------------------------------------------------------------+    |
|          |  | 10:00 AM | Review cabinet delivery schedule            | [...]  |   |
|          |  |          | Related: Order cabinets (Task Item)                |   |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  DECEMBER 26                                                           |
|          |  +---------------------------------------------------------------+    |
|          |  | 9:00 AM  | Weekly project status meeting               | [...]  |   |
|          |  |          | Recurring: Weekly                                  |   |
|          |  +---------------------------------------------------------------+    |
|          |  | 3:00 PM  | Invoice due: ABC Plumbing #4521             | [...]  |   |
|          |  |          | Related: Invoice #4521 (Riverside)                 |   |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  PAST (sent)                                              [Show/Hide] |
|          |  +---------------------------------------------------------------+    |
|          |  | Dec 20   | Check foundation cure progress              | Sent   |   |
|          |  | Dec 18   | Order rebar for footings                    | Sent   |   |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
+-----------------------------------------------------------------------------------+
```

---

## Filter Bar

| Filter | Field | Options |
|--------|-------|---------|
| Status | status | Pending, Sent, Cancelled, All |
| Date Range | remind_at | Today, This Week, This Month, All, Custom |

---

## Reminder Row

| Element | Description |
|---------|-------------|
| Time | remind_at time |
| Title | reminder title |
| Related | linked item (task, milestone, invoice) or "Custom" |
| Actions | Edit, Cancel, Delete |

---

## Grouping

Reminders grouped by date:
- Today
- Tomorrow
- Future dates (grouped by day)
- Past/Sent (collapsed)

---

## API Needed

### List Reminders

```
GET /reminders?status=pending&from_date=X&to_date=X&limit=50&offset=0

Response:
{
  data: [{
    id: uuid,
    title: string,
    description: string,
    remind_at: timestamp,
    recurrence: enum,
    status: enum,
    related_type: string,  // task_item, milestone, invoice, schedule_task, custom
    related_id: uuid,
    related_name: string,  // Populated from related entity
    project: { id, name },
    sent_at: timestamp
  }],
  total: number
}
```

---

## Add/Edit Reminder Modal

```
+------------------------------------------------------------------+
|  Add Reminder                                             [X]    |
+------------------------------------------------------------------+
|                                                                  |
|  Title *                                                         |
|  [_____________________________________________________]         |
|                                                                  |
|  Description                                                     |
|  [_____________________________________________________]         |
|                                                                  |
|  Remind At *                                                     |
|  Date: [____________]    Time: [__:__ v]                         |
|                                                                  |
|  Recurrence                                                      |
|  ( ) One-time                                                    |
|  ( ) Daily                                                       |
|  ( ) Weekly                                                      |
|  ( ) Monthly                                                     |
|                                                                  |
|  Link to (optional)                                              |
|  Type: [Select type...        v]                                 |
|  Item: [Select item...        v]    (populated based on type)    |
|                                                                  |
|  Project (optional)                                              |
|  [Select project...                                         v]   |
|                                                                  |
|                                      [Cancel]  [Save Reminder]   |
+------------------------------------------------------------------+
```

### Form Fields

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| title | string | Yes | Max 255 chars |
| description | string | No | Text |
| remind_at | datetime | Yes | Must be in future |
| recurrence | enum | No | none (default), daily, weekly, monthly |
| related_type | enum | No | task_item, milestone, invoice, schedule_task, custom |
| related_id | uuid | No | Required if related_type set (except custom) |
| project_id | uuid | No | Valid project |
| organization_id | uuid | No | Defaults to user's default org |

### API Needed

```
POST /reminders
Body: {
  title: string,
  remind_at: timestamp,
  description?: string,
  recurrence?: string,
  related_type?: string,
  related_id?: uuid,
  project_id?: uuid,
  organization_id?: uuid
}

PUT /reminders/:id
Body: { title?, remind_at?, recurrence?, ... }

DELETE /reminders/:id
```

---

## Reminder Detail (Expanded)

```
+------------------------------------------------------------------+
|  Call permit office about inspection                [Edit] [X]   |
+------------------------------------------------------------------+
|                                                                  |
|  When: December 23, 2024 at 9:00 AM                              |
|  Status: Pending                                                 |
|  Recurrence: One-time                                            |
|                                                                  |
|  Description:                                                    |
|  Call the city permit office to schedule the electrical          |
|  rough-in inspection. Ask for John in the inspections dept.      |
|                                                                  |
|  Related To:                                                     |
|  Milestone: Electrical Rough-in Inspection                       |
|  Project: Riverside Apartments                                   |
|                                                                  |
|  +------------------------------------------------------------+  |
|  |         [Cancel Reminder]        [Mark as Sent]            |  |
|  +------------------------------------------------------------+  |
|                                                                  |
+------------------------------------------------------------------+
```

### API Needed

```
GET /reminders/:id

Response:
{
  id, title, description, remind_at, recurrence, status,
  related_type, related_id,
  related: { id, name, ... },  // Populated related entity
  project: { id, name },
  organization: { id, name },
  sent_at, notification_id,
  created_at, updated_at
}

PATCH /reminders/:id/cancel
Response: { ...reminder with status: 'cancelled' }
```

---

## Auto-Generated Reminders

System can auto-create reminders based on:

| Source | Trigger | Default Time |
|--------|---------|--------------|
| Task Item due | due_date set | 24 hours before |
| Milestone approaching | target_date set | 7 days and 1 day before |
| Invoice due | due_date set | 7 days and 1 day before |
| Schedule task due | planned_end_date | 24 hours before |

User can customize or disable auto-reminders in Settings.

# Notifications

Full notification history with filtering.

Route: `/notifications`

---

## Full Page Layout

```
+-----------------------------------------------------------------------------------+
| [Logo] HarvestIQ     [_____Search_____]              [AI] [Bell(3)] [Avatar]      |
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  NOTIFICATIONS                                        [Mark All Read]  |
| Projects |                                                                        |
| Tasks    |  +---------------------------------------------------------------+    |
|          |  | Filter: [All v]   Status: [Unread v]                          |    |
|          |  +---------------------------------------------------------------+    |
| ──────── |                                                                        |
| Team     |  TODAY                                                                 |
| Settings |  +---------------------------------------------------------------+    |
|          |  |[*]| Task assigned to you                           | 2 hrs ago|   |
|          |  |   | "Order kitchen cabinets" - Riverside Apartments           |   |
|          |  +---------------------------------------------------------------+    |
|          |  |[*]| Invoice due soon                                | 5 hrs ago|   |
|          |  |   | Invoice #4521 from ABC Plumbing due in 3 days            |   |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  YESTERDAY                                                             |
|          |  +---------------------------------------------------------------+    |
|          |  |[ ]| Milestone approaching                          | Yesterday|   |
|          |  |   | "Framing Complete" due Dec 28 - Riverside Apartments     |   |
|          |  +---------------------------------------------------------------+    |
|          |  |[ ]| Reminder                                        | Yesterday|   |
|          |  |   | Follow up with electrician on quote                      |   |
|          |  +---------------------------------------------------------------+    |
|          |  |[ ]| Task overdue                                    | Yesterday|   |
|          |  |   | "Review subcontractor bids" was due Dec 20              |   |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  DECEMBER 20                                                           |
|          |  +---------------------------------------------------------------+    |
|          |  |[ ]| Budget alert                                    | Dec 20  |   |
|          |  |   | Framing costs exceed budget by 10% on Riverside Dev      |   |
|          |  +---------------------------------------------------------------+    |
|          |  |[ ]| Task completed                                  | Dec 20  |   |
|          |  |   | "Frame inspection" marked complete by Sarah              |   |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  Showing 1-10 of 45                         [< Prev] [1] [2] [Next >]  |
|          |                                                                        |
+-----------------------------------------------------------------------------------+
```

---

## Filter Bar

| Filter | Field | Options |
|--------|-------|---------|
| Type | notification_type | All, Task Assigned, Task Due, Milestone, Invoice, Reminder, Budget Alert, System |
| Status | read_at | Unread, Read, All |

---

## Notification Row

| Element | Description |
|---------|-------------|
| Unread indicator | [*] dot for unread, [ ] for read |
| Title | notification title (from notification_type) |
| Body | notification body text |
| Timestamp | relative time (2 hrs ago) or date |
| Click action | Marks as read, navigates to action_url |

---

## Notification Types

| Type | Icon | Title Pattern |
|------|------|---------------|
| task_assigned | User icon | "Task assigned to you" |
| task_due | Clock icon | "Task due soon" / "Task overdue" |
| milestone_approaching | Flag icon | "Milestone approaching" |
| invoice_due | Receipt icon | "Invoice due soon" |
| reminder | Bell icon | "Reminder" |
| budget_alert | Warning icon | "Budget alert" |
| system | Info icon | Varies |

---

## API Needed

### List Notifications

```
GET /notifications?notification_type=X&read=false&limit=20&offset=0

Response:
{
  data: [{
    id: uuid,
    notification_type: enum,
    title: string,
    body: string,
    action_url: string,
    data: object,      // Additional context
    priority: enum,
    read_at: timestamp,
    created_at: timestamp
  }],
  total: number,
  unread_count: number
}
```

### Get Unread Count

```
GET /notifications/unread-count

Response:
{
  count: number
}
```

### Mark as Read

```
PATCH /notifications/:id/read

Response:
{
  id, read_at: timestamp, ...
}
```

### Mark All as Read

```
POST /notifications/read-all

Response:
{
  updated_count: number
}
```

### Delete Notification

```
DELETE /notifications/:id
```

---

## Notification Dropdown (Header)

Quick access from the bell icon in header.

```
+------------------------------------------+
| Notifications                   [See All]|
+------------------------------------------+
|[*] Task assigned to you                  |
|    "Order kitchen cabinets"              |
|    2 hours ago                           |
+------------------------------------------+
|[*] Invoice due soon                      |
|    Invoice #4521 due in 3 days           |
|    5 hours ago                           |
+------------------------------------------+
|[ ] Milestone approaching                 |
|    "Framing Complete" - Dec 28           |
|    Yesterday                             |
+------------------------------------------+
|         [Mark All as Read]               |
+------------------------------------------+
```

### API Needed (Dropdown)

```
GET /notifications?limit=5&read=false

// Returns latest 5 unread notifications
```

---

## Notification Actions

| Action | Behavior |
|--------|----------|
| Click notification | Mark as read + navigate to action_url |
| Hover → X button | Delete notification |
| Mark All Read | Mark all visible as read |

---

## Notification Delivery Tracking

Each notification can be delivered via multiple channels.

```
GET /notifications/:id

Response:
{
  ...notification fields,
  deliveries: [{
    id: uuid,
    channel: enum,      // email, sms, in_app
    status: enum,       // pending, sent, failed
    sent_at: timestamp,
    error_message: string
  }]
}
```

This is primarily for admin/debug purposes, not shown in normal UI.

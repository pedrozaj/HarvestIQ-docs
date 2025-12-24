# Task Items

Personal task list showing items assigned to current user across all projects.

Route: `/task-items`

---

## Full Page Layout

```
+-----------------------------------------------------------------------------------+
| [Logo] HarvestIQ     [_____Search_____]              [AI] [Bell(3)] [Avatar]      |
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  MY TASK ITEMS                                     [+ Add Task Item]   |
| Projects |                                                                        |
| Tasks    |  +---------------------------------------------------------------+    |
|          |  | View: [My Tasks v]  Status: [Pending v]  Priority: [All v]    |    |
|          |  | Project: [All v]   Due: [All v]                               |    |
| ──────── |  +---------------------------------------------------------------+    |
| Team     |                                                                        |
| Settings |  OVERDUE (2)                                                           |
|          |  +---------------------------------------------------------------+    |
|          |  |[!][ ]| Follow up on permit approval   | Riverside | Dec 18   |    |
|          |  |      | High Priority                                          |    |
|          |  +---------------------------------------------------------------+    |
|          |  |[!][ ]| Review subcontractor bids      | Oak Grove | Dec 20   |    |
|          |  |      | Medium Priority                                        |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  DUE THIS WEEK (3)                                                     |
|          |  +---------------------------------------------------------------+    |
|          |  |[ ]| Order kitchen cabinets            | Riverside | Dec 28   |    |
|          |  |   | High Priority                                             |    |
|          |  +---------------------------------------------------------------+    |
|          |  |[ ]| Schedule plumbing inspection      | Oak Grove | Dec 30   |    |
|          |  |   | Medium Priority                                           |    |
|          |  +---------------------------------------------------------------+    |
|          |  |[ ]| Confirm electrical delivery       | Riverside | Dec 31   |    |
|          |  |   | Low Priority                                              |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  UPCOMING (5)                                                          |
|          |  +---------------------------------------------------------------+    |
|          |  |[ ]| Review framing estimate           | Summit    | Jan 5    |    |
|          |  |[ ]| Meet with architect               | Lakefront | Jan 8    |    |
|          |  |[ ]| Order roofing materials           | Riverside | Jan 10   |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  COMPLETED                                            [Show/Hide]      |
|          |  +---------------------------------------------------------------+    |
|          |  |[x]| Review electrical estimate        | Riverside | Dec 20   |    |
|          |  |[x]| Schedule foundation inspection    | Oak Grove | Dec 15   |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
+-----------------------------------------------------------------------------------+
```

---

## Filter Bar

| Filter | Field | Options |
|--------|-------|---------|
| View | assigned_to | My Tasks (default), All Tasks (admin) |
| Status | status | Pending, In Progress, Completed, All |
| Priority | priority | Low, Medium, High, Urgent, All |
| Project | project_id | Dropdown of projects |
| Due | due_date | Overdue, Today, This Week, This Month, All |

---

## Task Item Row

| Element | Description |
|---------|-------------|
| Overdue indicator | Red [!] if past due date |
| Checkbox | Toggle complete/incomplete |
| Title | Task title (click to expand/edit) |
| Project | Project name (click to go to project) |
| Due Date | Due date |
| Priority | Visual indicator |

---

## Grouping

Tasks grouped by due status:
1. Overdue - past due date, not completed
2. Due This Week - due within 7 days
3. Upcoming - due later
4. Completed - collapsed by default

---

## API Needed

### List Task Items

```
GET /task-items?assigned_to=me&status=pending&priority=high&project_id=X&due_before=X&limit=50&offset=0

Response:
{
  data: [{
    id: uuid,
    title: string,
    description: string,
    status: enum,
    priority: enum,
    due_date: date,
    project: { id, name },
    assigned_to: { id, name },
    created_by: { id, name },
    completed_at: timestamp,
    completed_by: { id, name },
    created_at: timestamp
  }],
  total: number,
  counts: {
    overdue: number,
    due_this_week: number,
    upcoming: number,
    completed: number
  }
}
```

### Get Overdue Count

```
GET /task-items/overdue?limit=0  // Just need count

Response:
{
  data: [],
  total: number  // This is the overdue count
}
```

---

## Add/Edit Task Item Modal

```
+------------------------------------------------------------------+
|  Add Task Item                                            [X]    |
+------------------------------------------------------------------+
|                                                                  |
|  Title *                                                         |
|  [_____________________________________________________]         |
|                                                                  |
|  Description                                                     |
|  [_____________________________________________________]         |
|  [_____________________________________________________]         |
|                                                                  |
|  Project                             Assign To                   |
|  [Select project...          v]      [Select user...        v]   |
|                                                                  |
|  Due Date                            Priority                    |
|  [____________]                      [Medium                 v]  |
|                                                                  |
|  Link to Schedule Task (optional)                                |
|  [Select schedule task...                                   v]   |
|                                                                  |
|                                       [Cancel]  [Save Task]      |
+------------------------------------------------------------------+
```

### Form Fields

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| title | string | Yes | Max 255 chars |
| description | string | No | Text |
| project_id | uuid | No | Valid project in user's orgs |
| assigned_to | uuid | No | Valid user in same builder |
| due_date | date | No | - |
| priority | enum | No | low, medium (default), high, urgent |
| schedule_task_id | uuid | No | Valid task in selected project |

### API Needed

```
POST /task-items
Body: {
  title: string,
  organization_id: uuid,
  project_id?: uuid,
  assigned_to?: uuid,
  due_date?: date,
  priority?: string,
  schedule_task_id?: uuid,
  description?: string
}

PUT /task-items/:id
Body: { title?, status?, priority?, due_date?, assigned_to?, description? }

DELETE /task-items/:id
```

---

## Task Item Detail (Expanded Row or Side Panel)

```
+------------------------------------------------------------------+
|  Order kitchen cabinets                          [Edit] [Delete] |
+------------------------------------------------------------------+
|                                                                  |
|  Status: [Pending v]     Priority: [High v]                      |
|  Due: December 28, 2024                                          |
|                                                                  |
|  Project: Riverside Apartments                                   |
|  Assigned to: Joey Smith                                         |
|  Created by: Sarah Johnson on Dec 15, 2024                       |
|                                                                  |
|  Description:                                                    |
|  Contact ABC Cabinets to finalize order. Need to confirm         |
|  measurements before placing order. Budget: $15,000              |
|                                                                  |
|  Related Schedule Task: Kitchen Installation Phase               |
|                                                                  |
|  +------------------------------------------------------------+  |
|  |                    [Mark as Complete]                      |  |
|  +------------------------------------------------------------+  |
|                                                                  |
+------------------------------------------------------------------+
```

### API Needed

```
GET /task-items/:id

Response:
{
  id, title, description, status, priority, due_date,
  project: { id, name },
  organization: { id, name },
  assigned_to: { id, name },
  created_by: { id, name },
  completed_at,
  completed_by: { id, name },
  schedule_task: { id, name },
  created_at, updated_at
}

PATCH /task-items/:id/complete
Response: { ...updated task item with completed_at, completed_by }
```

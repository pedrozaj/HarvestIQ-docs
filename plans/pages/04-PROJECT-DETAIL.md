# Project Detail

Single project hub with tabbed navigation for all project data.

Route: `/projects/:id`

---

## Page Header

```
+-----------------------------------------------------------------------------------+
| [Logo] HarvestIQ     [_____Search_____]              [AI] [Bell(3)] [Avatar]      |
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  [< Back to Projects]                                                  |
| Projects |                                                                        |
| Tasks    |  RIVERSIDE DEVELOPMENT                          [Edit] [Archive] [...] |
|          |  24 Townhomes - 123 Main Street, Anytown, USA                          |
| ──────── |  Status: [Active v]                                                    |
| Team     |                                                                        |
| Settings |  +----------+ +----------+ +----------+ +----------+                   |
|          |  | Budget   | | Spent    | | Remaining| | Progress |                   |
|          |  | $8.5M    | | $5.5M    | | $3.0M    | | 65%      |                   |
|          |  +----------+ +----------+ +----------+ +----------+                   |
|          |                                                                        |
|          |  [Overview] [Schedule] [Budget] [Documents] [Payments] [Tasks]         |
|          |  ================================================================      |
```

---

## Overview Tab

Default view showing project summary.

```
|          |                                                                        |
|          |  PROJECT DETAILS                           NEXT MILESTONES             |
|          |  +-------------------------------+        +-------------------------+  |
|          |  | Type:     Townhomes           |        | Electrical Rough-in    |  |
|          |  | Units:    24                  |        | Dec 28, 2024           |  |
|          |  | Lot Size: 8.5 acres           |        +-------------------------+  |
|          |  | Start:    Mar 15, 2024        |        | Plumbing Inspection    |  |
|          |  | Target:   Jun 30, 2025        |        | Jan 5, 2025            |  |
|          |  +-------------------------------+        +-------------------------+  |
|          |                                                                        |
|          |  BUDGET BY CATEGORY                SCHEDULE PROGRESS                   |
|          |  +---------------------------+     +-------------------------------+   |
|          |  | [====] Plumbing    $45k   |     | Foundation    [==========] 100%|  |
|          |  | [===]  Electrical  $38k   |     | Framing       [========  ] 80% |  |
|          |  | [==]   Framing     $25k   |     | Electrical    [====      ] 40% |  |
|          |  | [=]    Roofing     $18k   |     | Plumbing      [===       ] 30% |  |
|          |  +---------------------------+     +-------------------------------+   |
|          |                                                                        |
|          |  RECENT ACTIVITY                                                       |
|          |  +---------------------------------------------------------------+    |
|          |  | Payment $5,200 added for plumbing              | 2 hours ago  |    |
|          |  | Document "Invoice #4521" uploaded              | Yesterday    |    |
|          |  | Task "Frame inspection" completed              | 2 days ago   |    |
|          |  +---------------------------------------------------------------+    |
```

### API Needed

```
GET /projects/:id

Response:
{
  id, name, address, description, status,
  unit_type,       // single_family, townhomes, condos, apartments
  units,           // number of units
  lot_size_acres,
  start_date, target_end_date,
  organization: { id, name },
  created_by: { id, name },
  created_at, updated_at
}

GET /projects/:id/summary

Response:
{
  total_budget: number,
  total_spent: number,
  remaining: number,
  progress_percent: number,
  spending_by_category: [{ category_name, amount }],
  phases_progress: [{ phase_name, percent_complete }],
  upcoming_milestones: [{ id, name, target_date }]
}
```

---

## Schedule Tab

### Sub-Navigation

```
|          |  [Overview] [Schedule] [Budget] [Documents] [Payments] [Tasks]         |
|          |  =============================================================         |
|          |                                                                        |
|          |  Schedule Views: [Task List] [Timeline/Gantt] [Milestones] [Calendar]  |
```

### Task List View

```
|          |  +---------------------------------------------------------------+    |
|          |  | Filters: Phase [All v]  Status [All v]  Assignee [All v]      |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                          [+ Add Task] |
|          |  +---------------------------------------------------------------+    |
|          |  | Task                | Phase      | Assignee | Due      | Status|   |
|          |  +---------------------------------------------------------------+    |
|          |  | Install conduit     | Electrical | Joey     | Dec 28   | [v]   |   |
|          |  | Pull wire           | Electrical | --       | Jan 2    | [ ]   |   |
|          |  |   depends on: Install conduit                                 |   |
|          |  | Rough-in inspection | Electrical | Sarah    | Jan 5    | [ ]   |   |
|          |  |   depends on: Pull wire                                       |   |
|          |  +---------------------------------------------------------------+    |
```

### API Needed (Tasks)

```
GET /projects/:id/tasks?phase_id=X&status=X&assigned_to=X&limit=50

Response:
{
  data: [{
    id, name, description, status, priority,
    planned_start_date, planned_end_date,
    actual_start_date, actual_end_date,
    phase: { id, name },
    assigned_to: { id, name },
    dependencies: [{ id, name, status }]
  }]
}

POST /projects/:id/tasks
Body: { name, phase_id?, planned_start_date?, planned_end_date?, assigned_to?, priority? }

PUT /tasks/:id
Body: { status?, actual_start_date?, actual_end_date?, ... }

POST /tasks/:id/dependencies
Body: { depends_on_task_id, dependency_type? }
```

### Timeline/Gantt View

```
|          |                                                                        |
|          |       Dec 2024                    Jan 2025                             |
|          |       15  20  25  30  |  5   10   15   20   25   30                    |
|          |  +---------------------------------------------------------------+    |
|          |  | FOUNDATION                                                    |    |
|          |  |   Excavation   [====]                                         |    |
|          |  |   Pour footings     [===]                                     |    |
|          |  |   Foundation cure        [=====]                              |    |
|          |  |---------------------------------------------------------------+    |
|          |  | FRAMING                                                       |    |
|          |  |   Floor joists               [====]                           |    |
|          |  |   Wall framing                    [========]                  |    |
|          |  |   Roof trusses                              [====]            |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  Legend: [====] Complete  [----] In Progress  [....] Not Started      |
|          |          [!!!!] Blocked   --> Dependency arrow                         |
```

### API Needed (Timeline)

```
GET /projects/:id/timeline

Response:
{
  phases: [{
    id, name, start_date, end_date,
    tasks: [{
      id, name, planned_start_date, planned_end_date,
      actual_start_date, actual_end_date, status,
      dependencies: [{ task_id, type }]
    }]
  }],
  milestones: [{ id, name, target_date, status }]
}
```

### Milestones View

```
|          |  +---------------------------------------------------------------+    |
|          |  | Milestone           | Target Date | Actual Date | Status      |    |
|          |  +---------------------------------------------------------------+    |
|          |  | Foundation Complete | Nov 15      | Nov 14      | Achieved    |    |
|          |  | Framing Complete    | Dec 20      | --          | Pending     |    |
|          |  | Electrical Rough-in | Dec 28      | --          | Pending     |    |
|          |  | Plumbing Rough-in   | Jan 5       | --          | Pending     |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                      [+ Add Milestone]|
```

### API Needed (Milestones)

```
GET /projects/:id/milestones

Response:
{
  data: [{
    id, name, description, target_date, actual_date, status,
    phase: { id, name }
  }]
}

POST /projects/:id/milestones
Body: { name, target_date, phase_id?, description? }
```

---

## Budget Tab

```
|          |  [Overview] [Schedule] [Budget] [Documents] [Payments] [Tasks]         |
|          |  =============================================================         |
|          |                                                                        |
|          |  BUDGET SUMMARY                                                        |
|          |  +-------------------+ +-------------------+ +-------------------+     |
|          |  | Estimated         | | Actual            | | Variance          |     |
|          |  | $850,000          | | $552,500          | | -$297,500 (35%)   |     |
|          |  +-------------------+ +-------------------+ +-------------------+     |
|          |                                                                        |
|          |  [Progress Bar: ================>                            65%]      |
|          |                                                                        |
|          |  BY CATEGORY                                          [+ Add Item]    |
|          |  +---------------------------------------------------------------+    |
|          |  | Category    | Estimated  | Actual     | Variance   | Status   |    |
|          |  +---------------------------------------------------------------+    |
|          |  | Plumbing    | $65,000    | $45,230    | -$19,770   | On Track |    |
|          |  | Electrical  | $58,000    | $38,500    | -$19,500   | On Track |    |
|          |  | Framing     | $120,000   | $125,800   | +$5,800    | Over     |    |
|          |  | Roofing     | $45,000    | $0         | -$45,000   | Pending  |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  BUDGET ITEMS                              Filter: [All Categories v] |
|          |  +---------------------------------------------------------------+    |
|          |  | Item                | Category   | Estimated | Actual   | [...]|   |
|          |  +---------------------------------------------------------------+    |
|          |  | Rough-in labor      | Plumbing   | $25,000   | $24,500  |      |   |
|          |  | Fixtures            | Plumbing   | $18,000   | $12,730  |      |   |
|          |  | Pipe materials      | Plumbing   | $22,000   | $8,000   |      |   |
|          |  +---------------------------------------------------------------+    |
```

### API Needed (Budget)

```
GET /projects/:id/budget

Response:
{
  total_estimated: number,
  total_actual: number,
  variance: number,
  variance_percent: number,
  by_category: [{
    category: { id, name },
    estimated: number,
    actual: number,
    variance: number
  }],
  items: [{
    id, name, description,
    category: { id, name },
    phase: { id, name },
    estimated_amount, actual_amount
  }]
}

POST /projects/:id/budget-items
Body: { name, category_id, estimated_amount, phase_id?, description? }

PUT /budget-items/:id
Body: { actual_amount?, estimated_amount?, ... }
```

---

## Documents Tab

```
|          |  [Overview] [Schedule] [Budget] [Documents] [Payments] [Tasks]         |
|          |  =============================================================         |
|          |                                                                        |
|          |  +---------------------------------------------------------------+    |
|          |  | Type: [All v]  Tags: [___________]  Search: [___________]     |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                     [Upload Files]    |
|          |                                                                        |
|          |  View: [Grid] [List]                                                   |
|          |                                                                        |
|          |  +----------------+ +----------------+ +----------------+              |
|          |  | [PDF Icon]     | | [PDF Icon]     | | [IMG Icon]     |              |
|          |  | Invoice #4521  | | Plumbing Est   | | Site Photo     |              |
|          |  | Invoice        | | Estimate       | | Photo          |              |
|          |  | Dec 20, 2024   | | Dec 15, 2024   | | Dec 18, 2024   |              |
|          |  | [Download]     | | [Download]     | | [Download]     |              |
|          |  +----------------+ +----------------+ +----------------+              |
|          |                                                                        |
|          |  +----------------+ +----------------+ +----------------+              |
|          |  | [DOC Icon]     | | [PDF Icon]     | | [XLS Icon]     |              |
|          |  | Contract.docx  | | Permit App     | | Budget.xlsx    |              |
|          |  | Contract       | | Permit         | | Other          |              |
|          |  | Dec 10, 2024   | | Nov 28, 2024   | | Nov 15, 2024   |              |
|          |  | [Download]     | | [Download]     | | [Download]     |              |
|          |  +----------------+ +----------------+ +----------------+              |
```

### API Needed (Documents)

```
GET /projects/:id/documents?document_type=X&tags=X&search=X&limit=20&offset=0

Response:
{
  data: [{
    id, name, description, document_type, file_size, mime_type,
    tags: [string],
    uploaded_by: { id, name },
    created_at
  }],
  total: number
}

POST /projects/:id/documents
Content-Type: multipart/form-data
Body: { file, document_type, description?, tags? }

Response: { id, name, ..., upload_url? }

GET /documents/:id/download
Response: { url: "signed-r2-url", expires_at }
```

---

## Payments Tab

```
|          |  [Overview] [Schedule] [Budget] [Documents] [Payments] [Tasks]         |
|          |  =============================================================         |
|          |                                                                        |
|          |  PAYMENT SUMMARY                                                       |
|          |  +-------------------+ +-------------------+ +-------------------+     |
|          |  | Total Paid        | | Outstanding       | | This Month        |     |
|          |  | $5,525,000        | | $452,300          | | $382,000          |     |
|          |  +-------------------+ +-------------------+ +-------------------+     |
|          |                                                                        |
|          |  +---------------------------------------------------------------+    |
|          |  | Category: [All v]  Date: [From] - [To]  Method: [All v]       |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                    [+ Record Payment] |
|          |                                                                        |
|          |  +---------------------------------------------------------------+    |
|          |  | Date       | Description        | Category   | Amount  | Ref# |    |
|          |  +---------------------------------------------------------------+    |
|          |  | Dec 20     | Plumbing rough-in  | Plumbing   | $52,000 | 1234 |    |
|          |  | Dec 18     | Electrical panel   | Electrical | $38,000 | 5678 |    |
|          |  | Dec 15     | Framing labor      | Framing    | $125,000| 1233 |    |
|          |  +---------------------------------------------------------------+    |
```

### API Needed (Payments)

```
GET /projects/:id/payments?category_id=X&from_date=X&to_date=X&limit=20

Response:
{
  summary: {
    total_paid: number,
    outstanding: number,
    this_month: number
  },
  data: [{
    id, amount, payment_date, payment_method, reference_number, description,
    invoice: { id, invoice_number },
    budget_item: { id, name, category_name },
    created_by: { id, name }
  }]
}

POST /projects/:id/payments
Body: {
  amount, payment_date, description,
  payment_method?, reference_number?,
  invoice_id?, budget_item_id?
}
```

---

## Tasks Tab (Task Items)

Project-specific actionable items (separate from schedule tasks).

```
|          |  [Overview] [Schedule] [Budget] [Documents] [Payments] [Tasks]         |
|          |  =============================================================         |
|          |                                                                        |
|          |  +---------------------------------------------------------------+    |
|          |  | Status: [All v]  Priority: [All v]  Assignee: [All v]         |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                     [+ Add Task Item] |
|          |                                                                        |
|          |  +---------------------------------------------------------------+    |
|          |  |[ ]| Order kitchen cabinets        | High   | Joey  | Dec 28   |    |
|          |  |[ ]| Schedule plumbing inspection  | Medium | Sarah | Dec 30   |    |
|          |  |[x]| Review electrical estimate    | Low    | Joey  | Dec 20   |    |
|          |  |[ ]| Follow up on permit           | High   | --    | Jan 2    |    |
|          |  +---------------------------------------------------------------+    |
```

### API Needed (Task Items)

```
GET /projects/:id/task-items?status=X&priority=X&assigned_to=X

Response:
{
  data: [{
    id, title, description, status, priority, due_date,
    assigned_to: { id, name },
    created_by: { id, name },
    completed_at, completed_by
  }]
}

POST /task-items
Body: { title, project_id, organization_id, due_date?, priority?, assigned_to?, description? }

PUT /task-items/:id
Body: { status?, ... }
```

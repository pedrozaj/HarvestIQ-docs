# Dashboard

Main landing page after login. At-a-glance view of all projects and actionable items.

Route: `/dashboard`

---

## Full Page Layout

```
+-----------------------------------------------------------------------------------+
| [Logo] HarvestIQ     [_____Search_____]              [AI] [Bell(3)] [Avatar]      |
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  DASHBOARD                                                             |
| Projects |                                                                        |
| Tasks    |  +-------------------+ +-------------------+ +-------------------+     |
|          |  | Active Projects   | | Total Budget      | | Total Spent       |     |
|          |  |        12         | |    $2,450,000     | |    $1,823,500     |     |
| ──────── |  +-------------------+ +-------------------+ +-------------------+     |
| Team     |                                                                        |
| Settings |  +-------------------+ +-------------------+ +-------------------+     |
|          |  | Overdue Tasks     | | Unpaid Invoices   | | Upcoming Miles.   |     |
|          |  |        5          | |    $45,230        | |        3          |     |
|          |  +-------------------+ +-------------------+ +-------------------+     |
|          |                                                                        |
|          |  MY TASK ITEMS                                           [View All]   |
|          |  +---------------------------------------------------------------+    |
|          |  | [ ] Order kitchen cabinets          | Riverside | Due: Dec 28 |    |
|          |  | [ ] Schedule plumbing inspection    | Oak Grove | Due: Dec 30 |    |
|          |  | [ ] Review electrical estimate      | Riverside | Due: Jan 2  |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  +-----------------------------+  +--------------------------------+   |
|          |  | UPCOMING REMINDERS          |  | AI INSIGHTS                    |   |
|          |  +-----------------------------+  +--------------------------------+   |
|          |  | Call permit office          |  | [!] Plumbing costs on          |   |
|          |  | Tomorrow, 9:00 AM           |  |     Riverside are 30% above    |   |
|          |  +-----------------------------+  |     your average               |   |
|          |  | Follow up with electrician  |  +--------------------------------+   |
|          |  | Dec 26, 2:00 PM             |  | [i] 3 tasks blocked in         |   |
|          |  +-----------------------------+  |     Oak Grove project          |   |
|          |                                   +--------------------------------+   |
|          |                                                                        |
|          |  RECENT ACTIVITY                                                       |
|          |  +---------------------------------------------------------------+    |
|          |  | Joey added payment $5,200 to Riverside        | 2 hours ago   |    |
|          |  | Sarah completed task "Frame inspection"       | 5 hours ago   |    |
|          |  | Invoice #4521 uploaded to Oak Grove           | Yesterday     |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  PROJECTS                                                 [View All]   |
|          |  +-------------------+ +-------------------+ +-------------------+     |
|          |  | Riverside         | | Oak Grove         | | Summit Heights    |     |
|          |  | 24 Townhomes      | | 30 Single-Family  | | 48 Condos         |     |
|          |  | [=====>    ] 65%  | | [==>       ] 30%  | | [========>] 85%   |     |
|          |  | Budget: $8.5M     | | Budget: $12M      | | Budget: $15M      |     |
|          |  | Next: Electrical  | | Next: Foundation  | | Next: Final Insp  |     |
|          |  +-------------------+ +-------------------+ +-------------------+     |
|          |                                                                        |
+-----------------------------------------------------------------------------------+
```

---

## Summary Cards

| Card | Data Source | Value |
|------|-------------|-------|
| Active Projects | Count of projects where status = 'active' | number |
| Total Budget | Sum of all budget_items.estimated_amount | currency |
| Total Spent | Sum of all payments.amount | currency |
| Overdue Tasks | Count of schedule_tasks where planned_end_date < today | number |
| Unpaid Invoices | Sum of invoices where status != 'paid' | currency |
| Upcoming Milestones | Count of milestones in next 30 days | number |

### API Needed

```
GET /dashboard/summary

Response:
{
  active_projects: number,
  total_budget: number,
  total_spent: number,
  overdue_tasks: number,
  unpaid_invoices_amount: number,
  upcoming_milestones: number
}
```

---

## My Task Items Section

Shows task items assigned to current user, sorted by due date.

| Column | Field |
|--------|-------|
| Checkbox | status toggle |
| Title | title |
| Project | project.name |
| Due Date | due_date |

### API Needed

```
GET /task-items?assigned_to=me&status=pending&limit=5&sort=due_date

Response:
{
  data: [{
    id, title, due_date, priority, status,
    project: { id, name }
  }],
  total: number
}
```

---

## Upcoming Reminders Section

| Field | Source |
|-------|--------|
| Title | reminders.title |
| Date/Time | reminders.remind_at |

### API Needed

```
GET /reminders?status=pending&limit=5&sort=remind_at

Response:
{
  data: [{ id, title, remind_at, related_type, related_id }]
}
```

---

## AI Insights Section

| Field | Source |
|-------|--------|
| Icon | severity (info, warning, critical) |
| Description | ai_insights.description |

### API Needed

```
GET /ai/insights?is_dismissed=false&limit=5

Response:
{
  data: [{
    id, insight_type, title, description, severity,
    project: { id, name }
  }]
}
```

---

## Recent Activity Section

| Field | Source |
|-------|--------|
| Description | Generated from activity_log fields |
| Timestamp | activity_log.created_at |

### API Needed

```
GET /activity?limit=10

Response:
{
  data: [{
    id, action, entity_type, entity_name,
    user: { id, name },
    created_at
  }]
}
```

---

## Project Cards Section

| Field | Source |
|-------|--------|
| Name | projects.name |
| Progress | Calculated from schedule completion |
| Budget | Sum of budget_items.estimated_amount |
| Next Milestone | Next milestone by target_date |

### API Needed

```
GET /projects?status=active&limit=6&include=progress,next_milestone

Response:
{
  data: [{
    id, name, status,
    progress_percent: number,
    total_budget: number,
    total_spent: number,
    next_milestone: { name, target_date }
  }]
}
```

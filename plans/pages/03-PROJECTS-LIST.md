# Projects List

Browse, filter, and search all multi-unit development projects.

Route: `/projects`

---

## Full Page Layout

```
+-----------------------------------------------------------------------------------+
| [Logo] HarvestIQ     [_____Search_____]              [AI] [Bell(3)] [Avatar]      |
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  PROJECTS                                          [+ New Project]     |
| Projects |                                                                        |
| Tasks    |  +---------------------------------------------------------------+    |
|          |  | Filters:                                                      |    |
| ──────── |  | Status: [All v]  Org: [All v]  Type: [All v]  [Search...]     |    |
| Team     |  +---------------------------------------------------------------+    |
| Settings |                                                                        |
|          |  +---------------------------------------------------------------+    |
|          |  | Name              | Units | Type         | Budget   | Prog   |    |
|          |  +---------------------------------------------------------------+    |
|          |  | Riverside         | 24    | Townhomes    | $8.5M    | 65%    |    |
|          |  | 123 Main Street   |       | Active       |          |        |    |
|          |  +---------------------------------------------------------------+    |
|          |  | Oak Grove         | 30    | Single-Family| $12M     | 30%    |    |
|          |  | 456 Oak Avenue    |       | Active       |          |        |    |
|          |  +---------------------------------------------------------------+    |
|          |  | Summit Heights    | 48    | Condos       | $15M     | 85%    |    |
|          |  | 789 Summit Blvd   |       | Active       |          |        |    |
|          |  +---------------------------------------------------------------+    |
|          |  | Lakefront         | 36    | Townhomes    | $10.8M   | 0%     |    |
|          |  | 321 Lake Drive    |       | Planning     |          |        |    |
|          |  +---------------------------------------------------------------+    |
|          |  | Cedar Heights     | 20    | Single-Family| $6.5M    | 30%    |    |
|          |  | 555 Cedar Lane    |       | On Hold      |          |        |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  Showing 1-5 of 12 projects                    [< Prev] [1] [2] [Next >]|
|          |                                                                        |
+-----------------------------------------------------------------------------------+
```

---

## Filter Bar

| Filter | Field | Options |
|--------|-------|---------|
| Status | status | All, Planning, Active, On Hold, Completed |
| Organization | organization_id | Dropdown of user's organizations |
| Type | unit_type | All, Single-Family, Townhomes, Condos, Apartments |
| Search | q | Text search on name, address |

---

## Table Columns

| Column | Field | Sortable |
|--------|-------|----------|
| Name + Address | name, address | Yes (name) |
| Units | units | Yes |
| Type + Status | unit_type, status | Yes |
| Budget | Calculated sum | Yes |
| Progress | Calculated % | Yes |

---

## Row Actions (on hover or click)

- Click row → Navigate to project detail
- Three-dot menu: Edit, Archive, Delete

---

## API Needed

### List Projects

```
GET /projects?status=active&organization_id=X&unit_type=X&q=search&limit=20&offset=0&sort=name&order=asc

Response:
{
  data: [{
    id: uuid,
    name: string,
    address: string,
    status: enum,
    unit_type: string,    // single_family, townhomes, condos, apartments
    units: number,        // Number of units in development
    total_budget: number,
    total_spent: number,
    progress_percent: number,
    organization: { id, name }
  }],
  total: number,
  limit: number,
  offset: number
}
```

### Create Project

```
POST /projects

Body:
{
  organization_id: uuid (required),
  name: string (required),
  units: number (required),         // How many units in this development
  unit_type: string (required),     // single_family, townhomes, condos, apartments
  address?: string,
  description?: string,
  lot_size_acres?: number,
  start_date?: date,
  target_end_date?: date
}

Response:
{
  id: uuid,
  ...all project fields
}
```

---

## New Project Modal

```
+------------------------------------------------------------------+
|  Create New Project                                       [X]    |
+------------------------------------------------------------------+
|                                                                  |
|  Organization *                                                  |
|  [Select organization...                               v]        |
|                                                                  |
|  Project Name *                                                  |
|  [_____________________________________________________]         |
|                                                                  |
|  Number of Units *                   Unit Type *                 |
|  [_______________]                   [Single-Family          v]  |
|                                                                  |
|  Address                                                         |
|  [_____________________________________________________]         |
|                                                                  |
|  Lot Size (acres)                                                |
|  [_______________]                                               |
|                                                                  |
|  Start Date                          Target End Date             |
|  [____________]                      [____________]              |
|                                                                  |
|  Description                                                     |
|  [_____________________________________________________]         |
|  [_____________________________________________________]         |
|                                                                  |
|                                    [Cancel]  [Create Project]    |
+------------------------------------------------------------------+
```

### Form Fields

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| organization_id | uuid | Yes | Must be user's org |
| name | string | Yes | Max 255 chars |
| units | number | Yes | Positive integer (1+) |
| unit_type | enum | Yes | single_family, townhomes, condos, apartments |
| address | string | No | - |
| lot_size_acres | number | No | Positive |
| start_date | date | No | - |
| target_end_date | date | No | >= start_date |
| description | string | No | - |

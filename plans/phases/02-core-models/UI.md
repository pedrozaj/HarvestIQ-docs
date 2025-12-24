# Phase 2: UI Pages

App shell, team management, and project pages.

---

## App Shell Layout

Main layout for authenticated pages.

```
+-----------------------------------------------------------------------------------+
| [Logo] HarvestIQ     [_____Search_____]              [AI] [Bell(3)] [Avatar]      |
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|                                                                        |
| Projects |                         [Main Content Area]                            |
| Tasks    |                                                                        |
|          |                                                                        |
| ──────── |                                                                        |
| Team     |                                                                        |
| Settings |                                                                        |
|          |                                                                        |
+-----------------------------------------------------------------------------------+
```

### Header Components

| Element | Purpose | Implementation |
|---------|---------|----------------|
| Logo | Home link | Link to /dashboard |
| Search | Global search | Opens modal (Phase 6) |
| AI | AI assistant | Opens panel (Phase 7) |
| Bell | Notifications | Dropdown (Phase 5) |
| Avatar | User menu | Dropdown with logout |

### Sidebar Navigation

| Item | Route | Icon |
|------|-------|------|
| Dashboard | /dashboard | LayoutDashboard |
| Projects | /projects | Building2 |
| Tasks | /task-items | CheckSquare |
| Team | /team | Users |
| Settings | /settings | Settings |

### Implementation

```typescript
// src/app/(dashboard)/layout.tsx
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex h-screen">
      <Sidebar />
      <div className="flex-1 flex flex-col">
        <Header />
        <main className="flex-1 overflow-auto bg-muted/30">
          {children}
        </main>
      </div>
    </div>
  );
}
```

---

## Projects List Page

Route: `/projects`

```
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  PROJECTS                                          [+ New Project]     |
| Projects |                                                                        |
| Tasks    |  [All Statuses v] [All Types v] [All Orgs v] [________Search______]    |
|          |                                                                        |
| ──────── |  +---------------------------------------------------------------+    |
| Team     |  | Name              | Units    | Type       | Status  | Actions |    |
| Settings |  +---------------------------------------------------------------+    |
|          |  | Riverside Apts    | 24 units | Townhomes  | Active  | [...]   |    |
|          |  | 123 River Road    |          |            |         |         |    |
|          |  +---------------------------------------------------------------+    |
|          |  | Oak Grove Homes   | 30 units | Single Fam | Planning| [...]   |    |
|          |  | 456 Oak Street    |          |            |         |         |    |
|          |  +---------------------------------------------------------------+    |
|          |  | Summit Plaza      | 48 units | Condos     | Active  | [...]   |    |
|          |  | 789 Summit Ave    |          |            |         |         |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  Showing 1-10 of 15                         [< Prev] [1] [2] [Next >]  |
|          |                                                                        |
+-----------------------------------------------------------------------------------+
```

### Filter Options

| Filter | Values |
|--------|--------|
| Status | All, Planning, Active, On Hold, Completed |
| Type | All, Single Family, Townhomes, Condos, Apartments |
| Organization | All, [list of orgs] |

### Table Columns

| Column | Data | Sortable |
|--------|------|----------|
| Name | name + address | Yes |
| Units | units + "units" | Yes |
| Type | unitType label | No |
| Status | status badge | Yes |
| Actions | menu | No |

### Row Actions

```
+------------------+
| View Details     |
| Edit Project     |
+------------------+
| Archive Project  |
+------------------+
```

---

## Create Project Modal

```
+------------------------------------------------------------------+
|  New Project                                                [X]   |
+------------------------------------------------------------------+
|                                                                   |
|  Project Name *                                                   |
|  [_______________________________________________________]        |
|                                                                   |
|  Organization *                                                   |
|  [Acme Builders - Main                                  v]        |
|                                                                   |
|  Address                                                          |
|  [_______________________________________________________]        |
|                                                                   |
|  Unit Type *                    Units *                           |
|  [Townhomes                v]   [24_____________]                  |
|                                                                   |
|  Lot Size (acres)                                                 |
|  [_______________________________________________________]        |
|                                                                   |
|  Start Date                     End Date                          |
|  [____/__/____]                 [____/__/____]                    |
|                                                                   |
|  Description                                                      |
|  [_______________________________________________________]        |
|  [_______________________________________________________]        |
|                                                                   |
|                                    [Cancel]  [Create Project]     |
+------------------------------------------------------------------+
```

### Form Fields

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| name | text | Yes | min 2 chars |
| organizationId | select | Yes | valid org |
| address | text | No | - |
| unitType | select | Yes | enum value |
| units | number | Yes | min 1 |
| lotSizeAcres | number | No | min 0 |
| startDate | date | No | valid date |
| endDate | date | No | after start |
| description | textarea | No | - |

---

## Project Detail Page

Route: `/projects/:id`

```
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  [< Back] Riverside Apartments                                         |
| Projects |                                                                        |
| Tasks    |  [Active]  24 Townhomes  |  123 River Road  |  Started Dec 1, 2024    |
|          |                                                                        |
| ──────── |  [Overview] [Schedule] [Budget] [Documents] [Payments] [Tasks]         |
| Team     |  ================================================================       |
| Settings |                                                                        |
|          |  +---------------------------+  +---------------------------+          |
|          |  | BUDGET                    |  | SCHEDULE                  |          |
|          |  | $450,000 / $500,000       |  | 12 / 24 tasks complete    |          |
|          |  | [=========----] 90%       |  | [======------] 50%        |          |
|          |  | $50,000 remaining         |  | 3 overdue                 |          |
|          |  +---------------------------+  +---------------------------+          |
|          |                                                                        |
|          |  +---------------------------+  +---------------------------+          |
|          |  | NEXT MILESTONE            |  | COST PER UNIT             |          |
|          |  | Framing Complete          |  | $18,750                   |          |
|          |  | Due Dec 15, 2024          |  | ($450,000 / 24 units)     |          |
|          |  +---------------------------+  +---------------------------+          |
|          |                                                                        |
|          |  RECENT ACTIVITY                                                       |
|          |  +---------------------------------------------------------------+    |
|          |  | Joey created task "Install HVAC"              2 hours ago     |    |
|          |  | Sarah updated budget item "Plumbing"          3 hours ago     |    |
|          |  | Mike completed task "Framing"                 1 day ago       |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
+-----------------------------------------------------------------------------------+
```

### Tab Navigation

| Tab | Route | Purpose |
|-----|-------|---------|
| Overview | /projects/:id | Summary dashboard |
| Schedule | /projects/:id/schedule | Tasks, timeline |
| Budget | /projects/:id/budget | Budget items |
| Documents | /projects/:id/documents | Files |
| Payments | /projects/:id/payments | Invoices, payments |
| Tasks | /projects/:id/tasks | Project task items |

---

## Team Management Page

Route: `/team` (Admin only)

```
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  TEAM MANAGEMENT                                   [+ Invite Member]   |
| Projects |                                                                        |
| Tasks    |  Organization: [Acme Builders - Main               v]                  |
|          |                                                                        |
| ──────── |  [Members]  [Pending Invitations]                                      |
| Team     |  ============================================================          |
| Settings |                                                                        |
|          |  MEMBERS (5)                                                           |
|          |  +---------------------------------------------------------------+    |
|          |  | Name            | Email                  | Role   | Actions   |    |
|          |  +---------------------------------------------------------------+    |
|          |  | Joey Smith      | joey@acme.com          | Admin  | [...]     |    |
|          |  | (you)           | Joined Dec 1, 2024     |        |           |    |
|          |  +---------------------------------------------------------------+    |
|          |  | Sarah Johnson   | sarah@acme.com         | Admin  | [...]     |    |
|          |  |                 | Joined Dec 5, 2024     |        |           |    |
|          |  +---------------------------------------------------------------+    |
|          |  | Mike Wilson     | mike@acme.com          | Member | [...]     |    |
|          |  |                 | Joined Dec 10, 2024    |        |           |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
+-----------------------------------------------------------------------------------+
```

### Member Actions

```
+------------------+
| Change Role      |
|   > Admin        |
|   > Member       |
+------------------+
| Remove from Team |
+------------------+
```

### Invite Member Modal

```
+------------------------------------------------------------------+
|  Invite Team Member                                         [X]   |
+------------------------------------------------------------------+
|                                                                   |
|  Email Address *                                                  |
|  [_______________________________________________________]        |
|                                                                   |
|  Role                                                             |
|  ( ) Admin - Full access, can manage team                         |
|  (x) Member - Access to assigned projects only                    |
|                                                                   |
|  Personal Message (optional)                                      |
|  [_______________________________________________________]        |
|                                                                   |
|  Invitation expires in 7 days.                                    |
|                                                                   |
|                                      [Cancel]  [Send Invitation]  |
+------------------------------------------------------------------+
```

---

## Dashboard Page (Shell)

Route: `/dashboard`

Placeholder until Phase 6. Basic structure:

```
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  DASHBOARD                                                             |
| Projects |                                                                        |
| Tasks    |  Welcome back, Joey!                                                   |
|          |                                                                        |
| ──────── |  +---------------------------+  +---------------------------+          |
| Team     |  | ACTIVE PROJECTS           |  | BUDGET OVERVIEW           |          |
| Settings |  | 5                         |  | $2.1M total               |          |
|          |  +---------------------------+  +---------------------------+          |
|          |                                                                        |
|          |  YOUR PROJECTS                                                         |
|          |  +---------------------------------------------------------------+    |
|          |  | [Project cards grid - links to /projects/:id]                 |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
+-----------------------------------------------------------------------------------+
```

---

## Components to Build

| Component | Location | Purpose |
|-----------|----------|---------|
| Sidebar | components/layout/Sidebar.tsx | Navigation |
| Header | components/layout/Header.tsx | Top bar |
| UserMenu | components/layout/UserMenu.tsx | Avatar dropdown |
| ProjectCard | components/projects/ProjectCard.tsx | Project summary |
| ProjectForm | components/forms/ProjectForm.tsx | Create/edit |
| MemberTable | components/team/MemberTable.tsx | Member list |
| InviteForm | components/forms/InviteForm.tsx | Invite modal |
| StatusBadge | components/ui/StatusBadge.tsx | Status display |
| DataTable | components/ui/DataTable.tsx | Reusable table |

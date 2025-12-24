# Global Elements

Persistent UI elements available across all pages.

---

## Header Bar

```
+-----------------------------------------------------------------------------------+
|  [Logo] HarvestIQ     [_____Global Search_____] [?]   [AI]  [Bell(3)]  [Avatar v] |
+-----------------------------------------------------------------------------------+
```

### Components

| Element | Behavior |
|---------|----------|
| Logo | Click → Dashboard |
| Global Search | Opens search modal, searches across all entities |
| AI Button | Opens floating AI chat panel |
| Bell + Badge | Unread notification count, opens notification dropdown |
| Avatar | User menu dropdown (Profile, Settings, Logout) |

---

## Sidebar Navigation

```
+----------------------+
|                      |
|  Dashboard           |
|  Projects            |
|  Task Items          |
|                      |
|  ──────────────      |
|                      |
|  Team         [Admin]|
|  Settings            |
|                      |
+----------------------+
```

### Nav Items

| Item | Route | Access |
|------|-------|--------|
| Dashboard | `/dashboard` | All |
| Projects | `/projects` | All |
| Task Items | `/task-items` | All |
| Team | `/team` | Admin only |
| Settings | `/settings` | All |

---

## Global Search Modal

```
+------------------------------------------------------------------+
|  Search: [_________________________]                    [X Close] |
+------------------------------------------------------------------+
|  Filter: ( ) All  ( ) Projects  ( ) Documents  ( ) Tasks         |
|          ( ) Invoices                                            |
+------------------------------------------------------------------+
|                                                                  |
|  Results:                                                        |
|  +------------------------------------------------------------+  |
|  | [Project Icon] Riverside Development                       |  |
|  | Project - 24 Units - 123 Main St - Active                  |  |
|  +------------------------------------------------------------+  |
|  | [Doc Icon] Invoice #4521.pdf                               |  |
|  | Document - Riverside Development - Invoice                 |  |
|  +------------------------------------------------------------+  |
|  | [Task Icon] Order kitchen cabinets                         |  |
|  | Task Item - Riverside Development - Due Dec 28             |  |
|  +------------------------------------------------------------+  |
|                                                                  |
+------------------------------------------------------------------+
```

### Search Parameters

| Field | Type | Description |
|-------|------|-------------|
| q | string | Search query |
| type | enum | Filter: all, projects, documents, tasks, invoices |
| limit | number | Results per type (default 5) |

### API Needed

```
GET /search?q=X&type=all&limit=5

Response:
{
  projects: [{ id, name, address, units, status }],
  documents: [{ id, name, project_name, document_type }],
  tasks: [{ id, title, project_name, status }],
  invoices: [{ id, invoice_number, amount, status }]
}
```

---

## Floating AI Chat Panel

```
                                        +---------------------------+
                                        | AI Assistant        [_][X]|
                                        +---------------------------+
                                        |                           |
                                        | [Bot]: How can I help?    |
                                        |                           |
                                        | [You]: How much spent on  |
                                        |        plumbing?          |
                                        |                           |
                                        | [Bot]: You've spent       |
                                        |        $45,230 on         |
                                        |        plumbing across    |
                                        |        all projects...    |
                                        |                           |
                                        +---------------------------+
                                        | [Type a message...] [Send]|
                                        +---------------------------+
```

### Behavior

- Collapsible/expandable
- Persists conversation within session
- Context-aware: if on project page, scopes queries to that project
- Can be minimized to just the AI button in header

### API Needed

```
POST /ai/conversations
POST /ai/conversations/:id/messages
  body: { message: string, context?: { projectId?: string } }
```

---

## Notification Dropdown

```
                                              +---------------------------+
                                              | Notifications             |
                                              +---------------------------+
                                              | [*] Task "Order cabinets" |
                                              |     assigned to you       |
                                              |     2 hours ago           |
                                              +---------------------------+
                                              | [ ] Invoice #4521 is due  |
                                              |     in 3 days             |
                                              |     Yesterday             |
                                              +---------------------------+
                                              | [ ] Milestone approaching:|
                                              |     Framing Complete      |
                                              |     2 days ago            |
                                              +---------------------------+
                                              | [View All Notifications]  |
                                              +---------------------------+
```

### API Needed

```
GET /notifications?read=false&limit=10
GET /notifications/unread-count
PATCH /notifications/:id/read
```

---

## User Menu Dropdown

```
                                                      +------------------+
                                                      | Joey Smith       |
                                                      | joey@builder.com |
                                                      +------------------+
                                                      | Profile          |
                                                      | Settings         |
                                                      | ───────────────  |
                                                      | Logout           |
                                                      +------------------+
```

### API Needed

```
GET /users/me
POST /auth/logout
```

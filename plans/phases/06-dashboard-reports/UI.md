# Phase 6: UI Pages

Dashboard, search, and settings interfaces.

---

## Dashboard Page

Route: `/dashboard`

```
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  DASHBOARD                                                             |
| Projects |                                                                        |
| Tasks    |  +------------------+ +------------------+ +------------------+        |
|          |  | ACTIVE PROJECTS  | | TOTAL BUDGET     | | OVERDUE TASKS    |        |
| ──────── |  | 5                | | $2.1M            | | 3                |        |
| Team     |  +------------------+ +------------------+ +------------------+        |
| Settings |  +------------------+ +------------------+ +------------------+        |
|          |  | UPCOMING         | | UNPAID INVOICES  | | THIS MONTH       |        |
|          |  | MILESTONES: 4    | | $45,000          | | SPENT: $125K     |        |
|          |  +------------------+ +------------------+ +------------------+        |
|          |                                                                        |
|          |  +------------------------------+ +------------------------------+      |
|          |  | MY TASK ITEMS                | | UPCOMING REMINDERS           |      |
|          |  | [ ] Review permit (Dec 23)   | | Call permit office (Dec 24)  |      |
|          |  | [ ] Submit report (Dec 25)   | | Weekly review (Dec 26)       |      |
|          |  | [View All]                   | | [View All]                   |      |
|          |  +------------------------------+ +------------------------------+      |
|          |                                                                        |
|          |  AI INSIGHTS                                               [View All]  |
|          |  +---------------------------------------------------------------+    |
|          |  | [!] Plumbing costs on Riverside are 20% above average        |[x] |
|          |  | [i] Oak Grove completed framing 5 days ahead of schedule     |[x] |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  YOUR PROJECTS                                                         |
|          |  +------------------+ +------------------+ +------------------+        |
|          |  | Riverside Apts   | | Oak Grove Homes  | | Summit Plaza     |        |
|          |  | 24 Townhomes     | | 30 Single Family | | 48 Condos        |        |
|          |  | [Active]         | | [Active]         | | [Planning]       |        |
|          |  | Budget: 85%      | | Budget: 62%      | | Budget: 0%       |        |
|          |  | Schedule: 50%    | | Schedule: 78%    | | Schedule: 0%     |        |
|          |  | Next: Framing    | | Next: Drywall    | | Next: Permits    |        |
|          |  +------------------+ +------------------+ +------------------+        |
|          |                                                                        |
|          |  RECENT ACTIVITY                                                       |
|          |  +---------------------------------------------------------------+    |
|          |  | Joey created task "Install HVAC" on Riverside     2 hours ago |    |
|          |  | Sarah completed milestone "Framing" on Oak Grove  3 hours ago |    |
|          |  | Mike uploaded invoice "Plumbing Dec.pdf"          5 hours ago |    |
|          |  +---------------------------------------------------------------+    |
+-----------------------------------------------------------------------------------+
```

---

## Global Search Modal

Triggered by clicking search in header or pressing Cmd+K.

```
+------------------------------------------------------------------+
|                                                                   |
|  [______Search projects, tasks, documents..._______]              |
|                                                                   |
+------------------------------------------------------------------+
|                                                                   |
|  PROJECTS                                                         |
|  +--------------------------------------------------------------+|
|  | Riverside Apartments      | Active    | 24 Townhomes        ||
|  | Oak Grove Homes           | Active    | 30 Single Family    ||
|  +--------------------------------------------------------------+|
|                                                                   |
|  TASKS                                                            |
|  +--------------------------------------------------------------+|
|  | Install HVAC              | Riverside | In Progress         ||
|  | Framing inspection        | Oak Grove | Not Started         ||
|  +--------------------------------------------------------------+|
|                                                                   |
|  DOCUMENTS                                                        |
|  +--------------------------------------------------------------+|
|  | Plumbing Invoice Dec.pdf  | Riverside | Invoice             ||
|  +--------------------------------------------------------------+|
|                                                                   |
|  Press Enter to search all, ↑↓ to navigate, Esc to close         |
+------------------------------------------------------------------+
```

---

## Settings Page - Profile

Route: `/settings`

```
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  SETTINGS                                                              |
| Projects |                                                                        |
| Tasks    |  [Profile] [Security] [Notifications]                                  |
|          |  ================================                                      |
| ──────── |                                                                        |
| Team     |  PROFILE                                                               |
| Settings |                                                                        |
|          |  Name                                                                  |
|          |  [Joey Smith                                         ]                 |
|          |                                                                        |
|          |  Email                                                                 |
|          |  [joey@acmebuilders.com                             ] (cannot change)  |
|          |                                                                        |
|          |  Phone                                                                 |
|          |  [555-123-4567                                       ]                 |
|          |                                                                        |
|          |  Timezone                                                              |
|          |  [America/New_York                                  v]                 |
|          |                                                                        |
|          |                                              [Save Changes]            |
+-----------------------------------------------------------------------------------+
```

---

## Settings Page - Security

Route: `/settings/security`

```
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  SETTINGS                                                              |
| Projects |                                                                        |
| Tasks    |  [Profile] [Security] [Notifications]                                  |
|          |  ================================                                      |
| ──────── |                                                                        |
| Team     |  CHANGE PASSWORD                                                       |
| Settings |                                                                        |
|          |  Current Password                                                      |
|          |  [••••••••                                           ]                 |
|          |                                                                        |
|          |  New Password                                                          |
|          |  [••••••••                                           ]                 |
|          |  Min 8 chars, 1 uppercase, 1 lowercase, 1 number                       |
|          |                                                                        |
|          |  Confirm New Password                                                  |
|          |  [••••••••                                           ]                 |
|          |                                                                        |
|          |                                           [Change Password]            |
|          |                                                                        |
|          |  ─────────────────────────────────────────────────────                 |
|          |                                                                        |
|          |  SESSIONS                                                              |
|          |  Last login: Dec 23, 2024 at 9:15 AM                                   |
|          |  IP: 192.168.1.1                                                       |
|          |                                                                        |
+-----------------------------------------------------------------------------------+
```

---

## Components to Build

| Component | Purpose |
|-----------|---------|
| SummaryCard | Dashboard stat card |
| ProjectCard | Project summary tile |
| TaskItemPreview | Compact task item |
| InsightCard | AI insight with dismiss |
| ActivityItem | Activity feed row |
| SearchModal | Global search overlay |
| SearchResults | Grouped search results |
| ProfileForm | User profile form |
| PasswordForm | Password change form |

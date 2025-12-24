# Phase 3: Feature Implementation

User-facing functionality built on the data models.

---

## Dashboard

### Multi-Project Overview
- Summary cards showing active projects count, total budget, total spent
- Quick stats: upcoming milestones, overdue tasks, unpaid invoices
- Recent activity feed (from activity_log table)
- Pending task items assigned to current user
- Upcoming reminders

### Project Cards
- Project name, status, progress indicator
- Budget vs actual percentage
- Next milestone date
- Click to navigate to project detail

---

## Project Management

### Project List
- Paginated list with filters (status, organization)
- Search by name/address
- Sort by name, created date, status
- Quick status indicators

### Project Detail View
- Header: name, address, status badge, dates
- Tab navigation: Overview, Schedule, Budget, Documents, Payments, Tasks
- Progress indicators for schedule and budget
- Quick actions: edit, archive, export

### Project Comparison
- Select 2-3 projects to compare
- Side-by-side metrics: budget, spending, timeline
- Cost per unit comparison (calculated: total_spent / units)
- Efficiency metrics

---

## Schedule Management

### Phase Management
- Create, edit, reorder phases
- Drag-and-drop reordering
- Phase status tracking
- Date range display

### Task Management
- Task list with filters (status, assigned_to, priority, phase)
- Task creation with dependencies
- Inline status updates
- Assignment to team members
- Planned vs actual dates tracking

### Milestone Tracking
- Milestone list with status indicators
- Target date vs actual date
- Phase association
- Achievement/missed status

### Timeline Visualization
- Gantt chart view showing phases, tasks, milestones
- Task dependencies displayed as arrows
- Drag to adjust dates
- Color coding by status

### Calendar View
- Month/week view of tasks and milestones
- Click to view/edit details
- Filter by project, assignee

### Schedule Reports
- Overdue tasks list
- Upcoming deadlines (configurable days ahead)
- Schedule variance: planned vs actual comparison
- Blocked tasks with dependency details

---

## Budget Management

### Budget Overview
- Total estimated vs actual with variance
- Progress bar visualization
- Category breakdown chart (pie/bar)
- Phase allocation breakdown

### Budget Items
- List with category and phase filters
- Add/edit budget line items
- Estimated and actual amount entry
- Link to related payments

### Category Management
- Builder-level category definitions
- Create custom categories
- Deactivate unused categories

### Cost Analysis
- Spending by category report
- Spending by phase report
- Trend over time (line chart)
- Cross-project comparison
- Cost per unit analysis (calculated: total_spent / units)

### Budget Alerts
- Configure threshold alerts (percentage or amount)
- Alert when approaching/exceeding budget
- Per-project or builder-wide thresholds

---

## Document Management

### File Upload
- Drag and drop interface
- Multiple file selection
- Upload progress indicator
- Automatic type detection
- File size and type validation

### Document List
- Grid or list view toggle
- Filter by type (invoice, estimate, contract, permit, photo)
- Filter by tags
- Search by name, description, or content (extracted text)
- Sort by date, name, type

### Document Detail
- Preview for supported types (PDF, images)
- Metadata display (size, upload date, uploader)
- Tag management
- Download button with signed URL
- Link to related invoice/payment

### Tag Management
- Add/remove tags on documents
- Tag suggestions from existing tags
- Bulk tag operations

---

## Invoice Management

### Invoice List
- Filter by status (unpaid, partial, paid)
- Filter by category, date range
- Sort by due date, amount
- Overdue highlighting

### Invoice Detail
- Linked document preview
- Category information
- Amount and payment status
- Payment history for this invoice
- Add payment action

### Aging Report
- Group invoices by age (current, 30, 60, 90+ days)
- Total outstanding by project
- Total outstanding by category

---

## Payment Tracking

### Payment Entry
- Amount, date, payment method
- Link to invoice (optional)
- Link to budget category
- Description
- Reference number (check number, transaction ID)

### Payment History
- Chronological list with filters
- Filter by project, category, date range, method
- Search by reference number
- Export to CSV

### Outstanding Balances
- Summary of unpaid invoices
- By project breakdown
- By category breakdown

---

## Task Item Management

### Task Item List
- Personal view: assigned to current user
- Project view: all task items for project
- Organization view: all task items (admin)
- Filters: status, priority, due date, project

### Task Item Creation
- Title and description
- Due date picker
- Priority selection
- Project assignment (optional)
- User assignment (optional)
- Link to schedule task (optional)

### Task Item Actions
- Mark complete (records completed_at and completed_by)
- Change status
- Reassign to different user
- Set/modify reminder
- Edit details

### Overdue View
- List of overdue task items
- Sort by due date, priority
- Bulk actions

---

## Reminder Management

### Reminder List
- Upcoming reminders for current user
- Filter by status, date range
- Calendar integration view

### Reminder Creation
- Manual reminder creation
- Auto-generated from task items with due dates
- Auto-generated from milestone approaching dates
- Auto-generated from invoice due dates

### Reminder Settings
- One-time or recurring (daily, weekly, monthly)
- Custom remind-at time
- Link to related item

---

## Notification System

### Notification Bell
- Icon in header/navbar area
- Unread count badge
- Dropdown showing recent notifications
- Click to expand and navigate

### Notification Panel
- Full list with pagination
- Filter by type, read/unread
- Mark individual as read
- Mark all as read
- Delete notifications

### Notification Types

| Type | Default Channels | Trigger |
|------|------------------|---------|
| Task Assigned | In-app | Data change: task_items.assigned_to set |
| Task Due Soon | Email, In-app | Scheduled: 24 hours before due_date |
| Task Overdue | Email, In-app | Scheduled: when due_date passes |
| Milestone Approaching | Email, In-app | Scheduled: 7 days, 1 day before target_date |
| Invoice Due | Email | Scheduled: 7 days, 1 day before due_date |
| Reminder | Per user preference | Scheduled: at remind_at time |
| Budget Alert | Email, In-app | Triggered: when threshold exceeded |
| System | In-app | Various system events |

### Notification Preferences (User Settings)
- Enable/disable channels (email, SMS, in-app)
- Per-type channel selection
- Quiet hours configuration (optional)

### Notification Processing Flow

```
Trigger Event (data change or scheduled job)
    ↓
Create notification record in database
    ↓
Determine delivery channels from user preferences
    ↓
Create notification_deliveries records per channel
    ↓
Queue job for each delivery
    ↓
Background worker processes queue
    ↓
Send via channel (email/SMS/in-app)
    ↓
Update delivery status (sent/failed)
    ↓
Retry on failure (up to max_attempts)
```

---

## User Settings

### Profile
- Name, email, phone
- Password change
- Profile preferences

### Notification Preferences
- Channel toggles (email, SMS, in-app)
- Per-notification-type channel selection
- Quiet hours

---

## Team Management (Admin)

### Member List
- List organization members
- Role indicators
- Join date, last active

### Invitations
- Send invitation by email
- Pending invitations list
- Resend/revoke invitations

### Role Management
- Change member roles
- Remove members

---

## Frontend Routes

| Route | Purpose |
|-------|---------|
| `/` | Landing page / Login |
| `/register` | New builder registration |
| `/dashboard` | Multi-project overview |
| `/projects` | Project list |
| `/projects/:id` | Project detail (tabbed) |
| `/projects/:id/schedule` | Schedule management |
| `/projects/:id/budget` | Budget management |
| `/projects/:id/documents` | Document management |
| `/projects/:id/payments` | Payment tracking |
| `/projects/:id/tasks` | Project task items |
| `/task-items` | Personal task item list |
| `/reminders` | Reminder management |
| `/notifications` | Full notification history |
| `/settings` | User settings |
| `/settings/notifications` | Notification preferences |
| `/team` | Team management (admin) |
| `/ai` | AI chat interface |

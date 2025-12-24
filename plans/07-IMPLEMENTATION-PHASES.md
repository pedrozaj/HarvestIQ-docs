# Implementation Phases

Ordered breakdown of development work with dependencies.

---

## Phase Overview

```
Phase 1: Infrastructure ──┬──> Phase 2: Core Models ──┬──> Phase 3: Schedule/Budget
                          │                           │
                          │                           └──> Phase 4: Documents/Payments
                          │
                          └──────────────────────────────> Phase 5: Task Items/Notifications
                                                                      │
                                                                      v
                                                          Phase 6: Dashboard/Reports
                                                                      │
                                                                      v
                                                          Phase 7: AI Integration
```

---

## Phase 1: Infrastructure & Authentication

**Goal:** Establish foundation for all subsequent development.

### Backend Tasks

#### 1.1 Database Setup
- [ ] PostgreSQL database on Railway
- [ ] Connection pooling configuration
- [ ] Migration system setup (node-pg-migrate or similar)
- [ ] Seed data scripts for development

#### 1.2 Core Tables
```sql
-- Run in order (foreign key dependencies)
1. builders
2. users
3. organizations
4. organization_members
5. sessions (if not using JWT only)
```

#### 1.3 Authentication System
- [ ] Password hashing (Argon2id)
- [ ] JWT token generation and validation
- [ ] Refresh token rotation
- [ ] Login endpoint: `POST /auth/login`
- [ ] Registration endpoint: `POST /auth/register`
- [ ] Logout endpoint: `POST /auth/logout`
- [ ] Token refresh endpoint: `POST /auth/refresh`
- [ ] Password reset flow: `POST /auth/forgot-password`, `POST /auth/reset-password`

#### 1.4 Authorization Middleware
- [ ] `requireAuth` - Verify JWT, attach user to request
- [ ] `requireBuilder` - Ensure builder_id context
- [ ] `requireOrg` - Ensure organization membership
- [ ] `requireRole('admin')` - Role-based access check
- [ ] Row-level security helpers for builder isolation

#### 1.5 API Infrastructure
- [ ] Express 5 setup with async error handling
- [ ] Request validation middleware (zod or joi)
- [ ] Standardized response format
- [ ] Error handling middleware
- [ ] Rate limiting middleware
- [ ] CORS configuration
- [ ] Request logging

### Frontend Tasks

#### 1.6 Project Setup
- [ ] Next.js 15 app router structure
- [ ] Tailwind CSS configuration
- [ ] Component library setup (shadcn/ui recommended)
- [ ] API client with fetch wrapper
- [ ] Auth context provider

#### 1.7 Auth Pages
- [ ] Login page (`/login`)
- [ ] Registration page (`/register`)
- [ ] Forgot password page
- [ ] Reset password page
- [ ] Auth layout wrapper

#### 1.8 Protected Route System
- [ ] Auth middleware for protected routes
- [ ] Redirect logic for unauthenticated users
- [ ] Token storage (httpOnly cookies preferred)
- [ ] Auto-refresh token handling

### Deliverables
- User can register a new builder account
- User can login and receive JWT
- Protected routes redirect to login when unauthenticated
- Multi-tenant isolation enforced at API level

---

## Phase 2: Core Data Models

**Goal:** Establish project and organization entities.

**Depends on:** Phase 1 complete

### Backend Tasks

#### 2.1 Database Tables
```sql
-- Run in order
1. projects
2. activity_log
```

#### 2.2 Organization Endpoints
- [ ] `GET /organizations` - List user's organizations
- [ ] `GET /organizations/:id` - Get organization details
- [ ] `PUT /organizations/:id` - Update organization
- [ ] `GET /organizations/:id/members` - List members
- [ ] `POST /organizations/:id/members/invite` - Send invitation
- [ ] `PUT /organizations/:id/members/:userId/role` - Change role
- [ ] `DELETE /organizations/:id/members/:userId` - Remove member

#### 2.3 Invitation System
- [ ] `GET /invitations/validate` - Validate invitation token
- [ ] `POST /organizations/:id/members/accept` - Accept invitation
- [ ] `POST /organizations/:id/invitations/:id/resend` - Resend
- [ ] `DELETE /organizations/:id/invitations/:id` - Cancel

#### 2.4 Project Endpoints
- [ ] `GET /projects` - List projects (paginated, filtered)
- [ ] `POST /projects` - Create project
- [ ] `GET /projects/:id` - Get project details
- [ ] `PUT /projects/:id` - Update project
- [ ] `DELETE /projects/:id` - Soft delete project
- [ ] `GET /projects/:id/summary` - Calculated summary
- [ ] `GET /projects/compare` - Multi-project comparison

#### 2.5 Activity Logging
- [ ] Activity log insert helper
- [ ] `GET /projects/:id/activity` - Project activity feed
- [ ] `GET /activity` - Builder-wide activity feed

### Frontend Tasks

#### 2.6 App Shell
- [ ] Main layout with sidebar navigation
- [ ] Header with user menu, notifications bell, AI button
- [ ] Organization switcher (if multiple orgs)
- [ ] Responsive sidebar collapse

#### 2.7 Team Management Page (`/team`)
- [ ] Members list table
- [ ] Pending invitations tab
- [ ] Invite member modal
- [ ] Role change confirmation
- [ ] Remove member confirmation

#### 2.8 Projects List Page (`/projects`)
- [ ] Project cards/table view
- [ ] Status, unit type, organization filters
- [ ] Search by name/address
- [ ] Pagination
- [ ] Create project modal

#### 2.9 Project Detail Shell (`/projects/:id`)
- [ ] Project header with status, quick actions
- [ ] Tab navigation structure
- [ ] Overview tab with summary data

### Deliverables
- Admin can invite team members
- Team members can accept invitations
- Users can create and manage projects
- Project list with filtering and search
- Project detail page shell with overview

---

## Phase 3: Schedule & Budget

**Goal:** Implement core construction management features.

**Depends on:** Phase 2 complete

### Backend Tasks

#### 3.1 Database Tables
```sql
-- Run in order
1. budget_categories
2. schedule_phases
3. schedule_milestones
4. schedule_tasks
5. task_dependencies
6. budget_items
```

#### 3.2 Budget Category Endpoints
- [ ] `GET /budget-categories` - List builder's categories
- [ ] `POST /budget-categories` - Create category
- [ ] `PUT /budget-categories/:id` - Update category
- [ ] `DELETE /budget-categories/:id` - Deactivate category

#### 3.3 Schedule Phase Endpoints
- [ ] `GET /projects/:id/phases` - List phases
- [ ] `POST /projects/:id/phases` - Create phase
- [ ] `PUT /projects/:id/phases/:phaseId` - Update phase
- [ ] `DELETE /projects/:id/phases/:phaseId` - Delete phase
- [ ] `PUT /projects/:id/phases/reorder` - Reorder phases

#### 3.4 Schedule Task Endpoints
- [ ] `GET /projects/:id/tasks` - List tasks (filtered)
- [ ] `POST /projects/:id/tasks` - Create task
- [ ] `GET /projects/:id/tasks/:taskId` - Get task details
- [ ] `PUT /projects/:id/tasks/:taskId` - Update task
- [ ] `DELETE /projects/:id/tasks/:taskId` - Delete task
- [ ] `POST /projects/:id/tasks/:taskId/dependencies` - Add dependency
- [ ] `DELETE /projects/:id/tasks/:taskId/dependencies/:depId` - Remove dependency

#### 3.5 Milestone Endpoints
- [ ] `GET /projects/:id/milestones` - List milestones
- [ ] `POST /projects/:id/milestones` - Create milestone
- [ ] `PUT /projects/:id/milestones/:id` - Update milestone
- [ ] `DELETE /projects/:id/milestones/:id` - Delete milestone

#### 3.6 Timeline Endpoint
- [ ] `GET /projects/:id/timeline` - Aggregated timeline data for Gantt

#### 3.7 Budget Item Endpoints
- [ ] `GET /projects/:id/budget` - List budget items
- [ ] `POST /projects/:id/budget` - Create budget item
- [ ] `PUT /projects/:id/budget/:itemId` - Update item
- [ ] `DELETE /projects/:id/budget/:itemId` - Delete item
- [ ] `GET /projects/:id/budget/summary` - Budget summary with variance

### Frontend Tasks

#### 3.8 Schedule Tab - Task List View
- [ ] Task list with status, assignee, dates
- [ ] Filter by phase, status, assignee, priority
- [ ] Inline status updates
- [ ] Create/edit task modal
- [ ] Dependency management UI

#### 3.9 Schedule Tab - Gantt/Timeline View
- [ ] Gantt chart component (use library: frappe-gantt, dhtmlx, or custom)
- [ ] Phases as grouped rows
- [ ] Tasks as bars with drag-to-resize
- [ ] Dependency arrows
- [ ] Milestone diamonds
- [ ] Color coding by status

#### 3.10 Schedule Tab - Milestones View
- [ ] Milestone list with target/actual dates
- [ ] Status indicators (upcoming, achieved, missed)
- [ ] Create/edit milestone modal

#### 3.11 Schedule Tab - Calendar View
- [ ] Month/week calendar component
- [ ] Tasks and milestones on dates
- [ ] Click to view/edit
- [ ] Filter by assignee

#### 3.12 Budget Tab
- [ ] Budget overview with totals, variance
- [ ] Category breakdown chart (pie or bar)
- [ ] Budget items list with filters
- [ ] Create/edit budget item modal
- [ ] Category management (builder-level)

### Deliverables
- Full schedule management with phases, tasks, milestones
- Task dependencies and timeline visualization
- Gantt chart with drag-and-drop
- Calendar view of scheduled items
- Budget tracking with categories and variance

---

## Phase 4: Documents & Payments

**Goal:** Implement file storage and financial tracking.

**Depends on:** Phase 3 complete (budget categories needed)

### Backend Tasks

#### 4.1 Database Tables
```sql
-- Run in order
1. documents
2. invoices
3. payments
```

#### 4.2 File Storage Setup
- [ ] Cloudflare R2 integration
- [ ] Signed URL generation for uploads
- [ ] Signed URL generation for downloads
- [ ] File type and size validation
- [ ] Organize by builder_id/project_id path

#### 4.3 Document Endpoints
- [ ] `GET /projects/:id/documents` - List documents
- [ ] `POST /projects/:id/documents/upload-url` - Get signed upload URL
- [ ] `POST /projects/:id/documents` - Create document record
- [ ] `GET /projects/:id/documents/:docId` - Get document details
- [ ] `PUT /projects/:id/documents/:docId` - Update metadata
- [ ] `DELETE /projects/:id/documents/:docId` - Soft delete
- [ ] `GET /projects/:id/documents/:docId/download` - Get download URL
- [ ] `PUT /projects/:id/documents/:docId/tags` - Manage tags

#### 4.4 Invoice Endpoints
- [ ] `GET /projects/:id/invoices` - List invoices
- [ ] `POST /projects/:id/invoices` - Create invoice
- [ ] `GET /projects/:id/invoices/:invId` - Get invoice details
- [ ] `PUT /projects/:id/invoices/:invId` - Update invoice
- [ ] `DELETE /projects/:id/invoices/:invId` - Delete invoice
- [ ] `GET /invoices/aging` - Aging report across projects

#### 4.5 Payment Endpoints
- [ ] `GET /projects/:id/payments` - List payments
- [ ] `POST /projects/:id/payments` - Record payment
- [ ] `GET /projects/:id/payments/:payId` - Get payment details
- [ ] `PUT /projects/:id/payments/:payId` - Update payment
- [ ] `DELETE /projects/:id/payments/:payId` - Delete payment
- [ ] `GET /payments/outstanding` - Outstanding balances

### Frontend Tasks

#### 4.6 Documents Tab
- [ ] Document list with grid/list toggle
- [ ] Filter by type, tags, search
- [ ] Drag-and-drop upload zone
- [ ] Upload progress indicator
- [ ] Document preview modal (PDF, images)
- [ ] Tag management UI
- [ ] Download button

#### 4.7 Invoices Tab (within Payments)
- [ ] Invoice list with status indicators
- [ ] Filter by status, category, date
- [ ] Overdue highlighting
- [ ] Create/edit invoice modal
- [ ] Link to document upload
- [ ] Payment history for invoice

#### 4.8 Payments Tab
- [ ] Payment list with filters
- [ ] Record payment modal
- [ ] Link payment to invoice (optional)
- [ ] Payment method tracking
- [ ] Export to CSV

#### 4.9 Invoice Aging Report
- [ ] Grouped by age (current, 30, 60, 90+ days)
- [ ] Totals by project and category
- [ ] Drill-down to individual invoices

### Deliverables
- Document upload to R2 with signed URLs
- Document preview and download
- Invoice creation linked to documents
- Payment recording with invoice linkage
- Aging report for accounts receivable

---

## Phase 5: Task Items & Notifications

**Goal:** Implement personal productivity and communication features.

**Depends on:** Phase 2 complete (can run parallel to Phase 3/4)

### Backend Tasks

#### 5.1 Database Tables
```sql
-- Run in order
1. task_items
2. reminders
3. notifications
4. notification_preferences
5. notification_deliveries
```

#### 5.2 Task Item Endpoints
- [ ] `GET /task-items` - List user's task items
- [ ] `POST /task-items` - Create task item
- [ ] `GET /task-items/:id` - Get task item details
- [ ] `PUT /task-items/:id` - Update task item
- [ ] `DELETE /task-items/:id` - Delete task item
- [ ] `PUT /task-items/:id/complete` - Mark complete

#### 5.3 Reminder Endpoints
- [ ] `GET /reminders` - List user's reminders
- [ ] `POST /reminders` - Create reminder
- [ ] `PUT /reminders/:id` - Update reminder
- [ ] `DELETE /reminders/:id` - Delete reminder
- [ ] `PUT /reminders/:id/dismiss` - Dismiss reminder

#### 5.4 Notification Endpoints
- [ ] `GET /notifications` - List notifications (paginated)
- [ ] `PATCH /notifications/:id/read` - Mark as read
- [ ] `POST /notifications/read-all` - Mark all as read
- [ ] `DELETE /notifications/:id` - Delete notification

#### 5.5 Notification Preferences
- [ ] `GET /users/me/notification-preferences` - Get preferences
- [ ] `PUT /users/me/notification-preferences` - Update preferences

#### 5.6 Background Job System
- [ ] pg-boss setup for job queue
- [ ] Reminder check job (runs every minute)
- [ ] Due date notification job (runs daily)
- [ ] Email sending worker
- [ ] Notification delivery worker

#### 5.7 Alert Thresholds
- [ ] `GET /alert-thresholds` - List thresholds
- [ ] `POST /alert-thresholds` - Create threshold
- [ ] `PUT /alert-thresholds/:id` - Update threshold
- [ ] `DELETE /alert-thresholds/:id` - Delete threshold
- [ ] Budget alert check job

### Frontend Tasks

#### 5.8 Task Items Page (`/task-items`)
- [ ] Personal task list
- [ ] Overdue grouping at top
- [ ] Filter by status, priority, project
- [ ] Create/edit task item modal
- [ ] Quick complete action
- [ ] Link to schedule task (optional)

#### 5.9 Project Tasks Tab
- [ ] Project-scoped task items list
- [ ] Same functionality as personal view

#### 5.10 Reminders Page (`/reminders`)
- [ ] Upcoming reminders list
- [ ] Calendar view option
- [ ] Create/edit reminder modal
- [ ] Dismiss action

#### 5.11 Notifications
- [ ] Notification bell in header
- [ ] Dropdown with recent notifications
- [ ] Unread count badge
- [ ] Full notifications page (`/notifications`)
- [ ] Mark as read actions

#### 5.12 Settings - Notifications (`/settings/notifications`)
- [ ] Channel toggles (email, in-app)
- [ ] Per-type preferences
- [ ] Quiet hours setting

### Deliverables
- Personal task items with completion tracking
- Reminder system with scheduled notifications
- In-app notification bell and history
- Email notifications for due dates and assignments
- User notification preferences

---

## Phase 6: Dashboard & Reports

**Goal:** Implement aggregated views and search.

**Depends on:** Phases 3, 4, 5 complete

### Backend Tasks

#### 6.1 Dashboard Endpoint
- [ ] `GET /dashboard` - Aggregated dashboard data
  - Active projects count
  - Total budget/spent
  - Upcoming milestones
  - Overdue tasks count
  - Unpaid invoices count
  - Recent activity

#### 6.2 Global Search Endpoint
- [ ] `GET /search` - Search across entities
  - Projects by name/address
  - Documents by name/content
  - Tasks by title
  - Invoices by number

#### 6.3 Report Endpoints
- [ ] `GET /projects/:id/reports/schedule` - Schedule variance report
- [ ] `GET /projects/:id/reports/budget` - Budget breakdown report
- [ ] `GET /reports/spending` - Cross-project spending trends

### Frontend Tasks

#### 6.4 Dashboard Page (`/dashboard`)
- [ ] Summary cards (projects, budget, tasks, invoices)
- [ ] Project cards grid
- [ ] Upcoming milestones list
- [ ] My task items list
- [ ] Upcoming reminders
- [ ] AI insights section with dismiss actions
- [ ] Recent activity feed

#### 6.5 Global Search Modal
- [ ] Search input in header
- [ ] Results grouped by type
- [ ] Keyboard navigation
- [ ] Recent searches

#### 6.6 Settings Page (`/settings`)
- [ ] Profile tab
- [ ] Security tab (password change)
- [ ] Notification preferences tab

### Deliverables
- Dashboard with multi-project overview
- Global search across all entities
- User profile and settings management
- Schedule and budget reports

---

## Phase 7: AI Integration

**Goal:** Implement conversational AI interface.

**Depends on:** Phase 6 complete

### Backend Tasks

#### 7.1 Database Tables
```sql
-- Run in order
1. ai_conversations
2. ai_messages
3. ai_feedback
4. ai_insights
5. ai_patterns
6. document_embeddings
```

#### 7.2 Vector Database Setup
- [ ] Enable pgvector extension
- [ ] Create embedding indexes

#### 7.3 AI Conversation Endpoints
- [ ] `GET /ai/conversations` - List conversations
- [ ] `POST /ai/conversations` - Start new conversation
- [ ] `GET /ai/conversations/:id` - Get conversation with messages
- [ ] `DELETE /ai/conversations/:id` - Delete conversation

#### 7.4 AI Messaging
- [ ] `POST /ai/conversations/:id/messages` - Send message, get response
- [ ] Claude API integration
- [ ] Intent classification
- [ ] Entity extraction
- [ ] Query strategy selection
- [ ] Database query execution
- [ ] Response generation
- [ ] Token usage tracking

#### 7.5 AI Feedback
- [ ] `POST /ai/feedback` - Submit feedback on response

#### 7.6 AI Insights
- [ ] `GET /ai/insights` - Get active insights
- [ ] `POST /ai/insights/:id/dismiss` - Dismiss insight
- [ ] Insight generation background job

#### 7.7 AI Suggestions
- [ ] `GET /ai/suggestions` - Get contextual suggestions

#### 7.8 Document Processing Pipeline
- [ ] Text extraction job (PDF parsing)
- [ ] Chunking algorithm
- [ ] Embedding generation
- [ ] Store in document_embeddings
- [ ] Vector similarity search

#### 7.9 Token Limit Enforcement
- [ ] Track monthly usage per builder
- [ ] Reject queries when limit exceeded
- [ ] Monthly reset job

### Frontend Tasks

#### 7.10 AI Chat Page (`/ai`)
- [ ] Conversation sidebar
- [ ] Chat message area
- [ ] Context selector (all projects / specific project)
- [ ] Message input with suggestions
- [ ] Response with sources display
- [ ] Thumbs up/down feedback
- [ ] Token limit warning/error states

#### 7.11 Floating AI Panel
- [ ] AI button in header
- [ ] Collapsible chat panel
- [ ] Context-aware (inherits current project)
- [ ] "Open Full Chat" link
- [ ] Minimize to button

#### 7.12 Dashboard AI Insights
- [ ] Insights cards on dashboard
- [ ] Dismiss action
- [ ] Link to relevant data

### Deliverables
- Full conversational AI interface
- Natural language queries for project data
- Document search with RAG
- Proactive insights generation
- Floating AI panel on all pages

---

## Implementation Notes

### Parallel Work Opportunities

```
Phase 1 ─────────────────────────────────────────>

         Phase 2 ─────────────────────────────────>

                  Phase 3 ────────────────────────>
                  Phase 4 ────────────────────────> (can run parallel)
                  Phase 5 ────────────────────────> (can run parallel)

                                   Phase 6 ────────>

                                            Phase 7 >
```

- Phases 3, 4, 5 can be developed in parallel by different developers
- Phase 6 requires data from 3, 4, 5
- Phase 7 requires Phase 6 complete

### Technology Decisions

| Component | Recommended |
|-----------|-------------|
| Database migrations | node-pg-migrate |
| API validation | zod |
| Job queue | pg-boss |
| Email sending | Resend or SendGrid |
| File storage | Cloudflare R2 |
| Gantt chart | frappe-gantt or dhtmlx-gantt |
| Calendar | react-big-calendar |
| PDF preview | react-pdf |
| Charts | recharts or chart.js |
| AI/LLM | Claude API |
| Embeddings | OpenAI embeddings API |

### Testing Strategy

| Phase | Testing Focus |
|-------|---------------|
| Phase 1 | Auth flows, token validation, middleware |
| Phase 2 | CRUD operations, authorization checks |
| Phase 3 | Dependency logic, date calculations |
| Phase 4 | File upload/download, payment math |
| Phase 5 | Job scheduling, notification delivery |
| Phase 6 | Aggregation accuracy, search relevance |
| Phase 7 | AI response quality, token tracking |

### Migration Order Reference

Complete table creation order respecting foreign keys:

```
1.  builders
2.  users
3.  organizations
4.  organization_members
5.  projects
6.  budget_categories
7.  schedule_phases
8.  schedule_milestones
9.  schedule_tasks
10. task_dependencies
11. budget_items
12. documents
13. invoices
14. payments
15. task_items
16. reminders
17. notifications
18. notification_preferences
19. notification_deliveries
20. alert_thresholds
21. activity_log
22. ai_conversations
23. ai_messages
24. ai_feedback
25. ai_insights
26. ai_patterns
27. document_embeddings
```

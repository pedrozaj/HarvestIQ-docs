# API Specification

Complete API reference aligned with UI wireframes. All endpoints require authentication unless noted.

Base URL: `https://harvestiq-backend-production.up.railway.app`

---

## Authentication

### POST /auth/register
Create new builder account with admin user.

**Request:**
```json
{
  "company_name": "string (required) - Builder/company name",
  "name": "string (required) - User's full name",
  "email": "string (required) - Valid email, unique",
  "phone": "string (optional)",
  "password": "string (required) - Min 8 characters"
}
```

**Response (201):**
```json
{
  "message": "Verification email sent"
}
```

**Errors:**
- 400: Validation error (invalid email, password too short)
- 409: Email already exists

---

### POST /auth/login
Authenticate user and receive tokens.

**Request:**
```json
{
  "email": "string (required)",
  "password": "string (required)"
}
```

**Response (200):**
```json
{
  "accessToken": "string - JWT valid 15 minutes",
  "user": {
    "id": "uuid",
    "name": "string",
    "email": "string",
    "role": "admin | member"
  }
}
```

**Cookies Set:**
- `refresh_token` - HTTP-only, Secure, SameSite=Strict, 7 days

**Errors:**
- 401: Invalid credentials
- 403: Email not verified
- 423: Account locked (too many failed attempts)

---

### POST /auth/logout
Revoke refresh token and end session.

**Request:** None (uses cookie)

**Response (200):**
```json
{
  "message": "Logged out successfully"
}
```

---

### POST /auth/refresh
Get new access token using refresh token.

**Request:** None (uses cookie)

**Response (200):**
```json
{
  "accessToken": "string"
}
```

**Errors:**
- 401: Invalid or expired refresh token
- 403: Subscription inactive

---

### POST /auth/forgot-password
Request password reset email.

**Request:**
```json
{
  "email": "string (required)"
}
```

**Response (200):**
```json
{
  "message": "If account exists, reset email sent"
}
```

---

### POST /auth/reset-password
Set new password with reset token.

**Request:**
```json
{
  "token": "string (required) - From email link",
  "password": "string (required) - Min 8 characters"
}
```

**Response (200):**
```json
{
  "message": "Password reset successful"
}
```

**Errors:**
- 400: Invalid or expired token

---

### POST /auth/verify-email
Verify email address with token.

**Request:**
```json
{
  "token": "string (required) - From email link"
}
```

**Response (200):**
```json
{
  "message": "Email verified successfully"
}
```

---

### POST /auth/resend-verification
Resend verification email.

**Request:**
```json
{
  "email": "string (required)"
}
```

**Response (200):**
```json
{
  "message": "Verification email sent"
}
```

---

## Current User

### GET /users/me
Get current user profile.

**Response (200):**
```json
{
  "id": "uuid",
  "name": "string",
  "email": "string",
  "phone": "string | null",
  "email_verified_at": "timestamp | null",
  "last_login_at": "timestamp | null",
  "notification_preferences": {
    "channels": { "email": true, "sms": false, "in_app": true },
    "types": {
      "task_assigned": ["in_app"],
      "task_due": ["email", "in_app"],
      "reminder": ["email", "sms", "in_app"]
    }
  },
  "builder": { "id": "uuid", "name": "string" },
  "organizations": [
    { "id": "uuid", "name": "string", "role": "admin | member" }
  ]
}
```

---

### PUT /users/me
Update current user profile.

**Request:**
```json
{
  "name": "string (optional)",
  "phone": "string (optional)"
}
```

**Response (200):** Updated user object

---

### PUT /users/me/password
Change password.

**Request:**
```json
{
  "current_password": "string (required)",
  "new_password": "string (required) - Min 8 characters"
}
```

**Response (200):**
```json
{
  "message": "Password changed successfully"
}
```

**Side Effects:** Invalidates all other sessions

**Errors:**
- 401: Current password incorrect

---

### PUT /users/me/notification-preferences
Update notification preferences.

**Request:**
```json
{
  "channels": {
    "email": "boolean",
    "sms": "boolean",
    "in_app": "boolean"
  },
  "types": {
    "task_assigned": ["in_app"],
    "task_due": ["email", "in_app"],
    "task_overdue": ["email", "sms", "in_app"],
    "milestone_approaching": ["email", "in_app"],
    "invoice_due": ["email"],
    "reminder": ["email", "sms", "in_app"],
    "budget_alert": ["email", "in_app"],
    "system": ["in_app"]
  }
}
```

**Response (200):** Updated preferences object

---

### GET /users/me/sessions
List active sessions.

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "device_info": "string",
      "created_at": "timestamp",
      "last_used_at": "timestamp",
      "is_current": "boolean"
    }
  ]
}
```

---

### DELETE /users/me/sessions/:id
Revoke specific session.

**Response (200):**
```json
{
  "message": "Session revoked"
}
```

---

### DELETE /users/me/sessions
Revoke all sessions except current.

**Response (200):**
```json
{
  "revoked_count": "number"
}
```

---

## Builder (Tenant)

### GET /builder
Get current builder details.

**Response (200):**
```json
{
  "id": "uuid",
  "name": "string",
  "subscription_status": "active | trial | cancelled | past_due",
  "subscription_plan": "free | pro | enterprise",
  "trial_ends_at": "timestamp | null",
  "storage_used_bytes": "number",
  "storage_limit_bytes": "number",
  "ai_tokens_used_month": "number",
  "ai_tokens_limit_month": "number"
}
```

---

### PUT /builder
Update builder settings. (Admin only)

**Request:**
```json
{
  "name": "string (optional)",
  "billing_email": "string (optional)"
}
```

**Response (200):** Updated builder object

---

## Organizations

### GET /organizations
List organizations.

**Query Parameters:**
- `limit` (number, default 20)
- `offset` (number, default 0)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "string",
      "member_count": "number",
      "project_count": "number",
      "created_at": "timestamp"
    }
  ],
  "total": "number",
  "limit": "number",
  "offset": "number"
}
```

---

### POST /organizations
Create organization. (Admin only)

**Request:**
```json
{
  "name": "string (required)"
}
```

**Response (201):**
```json
{
  "id": "uuid",
  "name": "string",
  "created_at": "timestamp"
}
```

---

### GET /organizations/:id
Get organization details.

**Response (200):**
```json
{
  "id": "uuid",
  "name": "string",
  "settings": "object",
  "member_count": "number",
  "project_count": "number",
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

---

### PUT /organizations/:id
Update organization. (Admin only)

**Request:**
```json
{
  "name": "string (optional)",
  "settings": "object (optional)"
}
```

**Response (200):** Updated organization object

---

### DELETE /organizations/:id
Soft delete organization. (Admin only)

**Response (200):**
```json
{
  "message": "Organization deleted"
}
```

---

## Organization Members

### GET /organizations/:id/members
List organization members.

**Query Parameters:**
- `role` (admin | member, optional)
- `is_active` (boolean, optional)
- `limit` (number, default 50)
- `offset` (number, default 0)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid - membership id",
      "user": {
        "id": "uuid",
        "name": "string",
        "email": "string"
      },
      "role": "admin | member",
      "joined_at": "timestamp",
      "is_active": "boolean"
    }
  ],
  "total": "number"
}
```

---

### POST /organizations/:id/members/invite
Send invitation. (Admin only)

**Request:**
```json
{
  "email": "string (required)",
  "role": "admin | member (required)",
  "message": "string (optional) - Personal note in email"
}
```

**Response (201):**
```json
{
  "id": "uuid",
  "email": "string",
  "role": "string",
  "expires_at": "timestamp"
}
```

**Errors:**
- 409: Already a member

---

### GET /organizations/:id/invitations
List pending invitations. (Admin only)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "email": "string",
      "role": "string",
      "invited_by": { "id": "uuid", "name": "string" },
      "created_at": "timestamp",
      "expires_at": "timestamp"
    }
  ]
}
```

---

### POST /organizations/:id/invitations/:inviteId/resend
Resend invitation. (Admin only)

**Response (200):**
```json
{
  "message": "Invitation resent",
  "expires_at": "timestamp"
}
```

---

### DELETE /organizations/:id/invitations/:inviteId
Cancel invitation. (Admin only)

**Response (200):**
```json
{
  "message": "Invitation cancelled"
}
```

---

### GET /invitations/validate
Validate invitation token. (No auth required)

**Query Parameters:**
- `token` (string, required)

**Response (200):**
```json
{
  "valid": "boolean",
  "email": "string",
  "organization": { "id": "uuid", "name": "string" },
  "invited_by": { "name": "string" },
  "expires_at": "timestamp"
}
```

---

### POST /organizations/:id/members/accept
Accept invitation.

**Request:**
```json
{
  "token": "string (required)"
}
```

**Response (200):**
```json
{
  "organization": { "id": "uuid", "name": "string" },
  "role": "string"
}
```

---

### PUT /organizations/:id/members/:userId/role
Change member role. (Admin only)

**Request:**
```json
{
  "role": "admin | member"
}
```

**Response (200):**
```json
{
  "message": "Role updated"
}
```

**Errors:**
- 400: Cannot demote last admin

---

### DELETE /organizations/:id/members/:userId
Remove member. (Admin only)

**Response (200):**
```json
{
  "message": "Member removed"
}
```

**Errors:**
- 400: Cannot remove yourself

---

## Projects

### GET /projects
List projects.

**Query Parameters:**
- `organization_id` (uuid, optional)
- `status` (planning | active | on_hold | completed, optional)
- `unit_type` (single_family | townhomes | condos | apartments, optional)
- `q` (string, search name/address)
- `sort` (name | created_at | units | status, default created_at)
- `order` (asc | desc, default desc)
- `limit` (number, default 20)
- `offset` (number, default 0)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "string",
      "address": "string | null",
      "status": "string",
      "unit_type": "string",
      "units": "number",
      "total_budget": "number",
      "total_spent": "number",
      "progress_percent": "number",
      "organization": { "id": "uuid", "name": "string" },
      "created_at": "timestamp"
    }
  ],
  "total": "number",
  "limit": "number",
  "offset": "number"
}
```

---

### POST /projects
Create project.

**Request:**
```json
{
  "organization_id": "uuid (required)",
  "name": "string (required)",
  "units": "number (required) - Number of units in development",
  "unit_type": "single_family | townhomes | condos | apartments (required)",
  "address": "string (optional)",
  "description": "string (optional)",
  "lot_size_acres": "number (optional)",
  "start_date": "date (optional)",
  "target_end_date": "date (optional)"
}
```

**Response (201):**
```json
{
  "id": "uuid",
  "name": "string",
  "address": "string | null",
  "description": "string | null",
  "status": "planning",
  "unit_type": "string",
  "units": "number",
  "lot_size_acres": "number | null",
  "start_date": "date | null",
  "target_end_date": "date | null",
  "organization": { "id": "uuid", "name": "string" },
  "created_by": { "id": "uuid", "name": "string" },
  "created_at": "timestamp"
}
```

---

### GET /projects/:id
Get project details.

**Response (200):**
```json
{
  "id": "uuid",
  "name": "string",
  "address": "string | null",
  "description": "string | null",
  "status": "planning | active | on_hold | completed",
  "unit_type": "single_family | townhomes | condos | apartments",
  "units": "number",
  "lot_size_acres": "number | null",
  "start_date": "date | null",
  "target_end_date": "date | null",
  "actual_end_date": "date | null",
  "organization": { "id": "uuid", "name": "string" },
  "created_by": { "id": "uuid", "name": "string" },
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

---

### PUT /projects/:id
Update project.

**Request:**
```json
{
  "name": "string (optional)",
  "address": "string (optional)",
  "description": "string (optional)",
  "status": "string (optional)",
  "unit_type": "string (optional)",
  "units": "number (optional)",
  "lot_size_acres": "number (optional)",
  "start_date": "date (optional)",
  "target_end_date": "date (optional)",
  "actual_end_date": "date (optional)"
}
```

**Response (200):** Updated project object

---

### DELETE /projects/:id
Soft delete project.

**Response (200):**
```json
{
  "message": "Project deleted"
}
```

---

### GET /projects/:id/summary
Get project statistics summary.

**Response (200):**
```json
{
  "total_budget": "number",
  "total_spent": "number",
  "remaining": "number",
  "progress_percent": "number",
  "cost_per_unit": "number",
  "spending_by_category": [
    { "category_id": "uuid", "category_name": "string", "amount": "number" }
  ],
  "phases_progress": [
    { "phase_id": "uuid", "phase_name": "string", "percent_complete": "number" }
  ],
  "upcoming_milestones": [
    { "id": "uuid", "name": "string", "target_date": "date" }
  ],
  "overdue_tasks": "number",
  "blocked_tasks": "number"
}
```

---

### GET /projects/compare
Compare multiple projects.

**Query Parameters:**
- `ids` (comma-separated uuids, required, max 5)

**Response (200):**
```json
{
  "projects": [
    {
      "id": "uuid",
      "name": "string",
      "units": "number",
      "unit_type": "string",
      "total_budget": "number",
      "total_spent": "number",
      "cost_per_unit": "number",
      "progress_percent": "number",
      "days_elapsed": "number",
      "days_remaining": "number"
    }
  ]
}
```

---

## Schedule - Phases

### GET /projects/:id/phases
List phases for project.

**Query Parameters:**
- `status` (not_started | in_progress | completed, optional)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "string",
      "description": "string | null",
      "start_date": "date | null",
      "end_date": "date | null",
      "status": "string",
      "sort_order": "number",
      "task_count": "number",
      "completed_task_count": "number"
    }
  ]
}
```

---

### POST /projects/:id/phases
Create phase.

**Request:**
```json
{
  "name": "string (required)",
  "description": "string (optional)",
  "start_date": "date (optional)",
  "end_date": "date (optional)",
  "sort_order": "number (optional) - Defaults to end"
}
```

**Response (201):** Phase object

---

### PUT /phases/:id
Update phase.

**Request:**
```json
{
  "name": "string (optional)",
  "description": "string (optional)",
  "start_date": "date (optional)",
  "end_date": "date (optional)",
  "status": "string (optional)"
}
```

**Response (200):** Updated phase object

---

### DELETE /phases/:id
Soft delete phase.

**Response (200):**
```json
{
  "message": "Phase deleted"
}
```

---

### PUT /projects/:id/phases/reorder
Reorder phases.

**Request:**
```json
{
  "phase_ids": ["uuid", "uuid", "uuid"]
}
```

**Response (200):**
```json
{
  "message": "Phases reordered"
}
```

---

## Schedule - Tasks

### GET /projects/:id/tasks
List tasks for project.

**Query Parameters:**
- `phase_id` (uuid, optional)
- `status` (not_started | in_progress | completed | blocked, optional)
- `assigned_to` (uuid, optional)
- `priority` (low | medium | high | urgent, optional)
- `limit` (number, default 50)
- `offset` (number, default 0)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "string",
      "description": "string | null",
      "status": "string",
      "priority": "string",
      "planned_start_date": "date | null",
      "planned_end_date": "date | null",
      "actual_start_date": "date | null",
      "actual_end_date": "date | null",
      "phase": { "id": "uuid", "name": "string" } | null,
      "assigned_to": { "id": "uuid", "name": "string" } | null,
      "dependencies": [
        { "id": "uuid", "name": "string", "status": "string" }
      ]
    }
  ],
  "total": "number"
}
```

---

### POST /projects/:id/tasks
Create task.

**Request:**
```json
{
  "name": "string (required)",
  "description": "string (optional)",
  "phase_id": "uuid (optional)",
  "assigned_to": "uuid (optional)",
  "priority": "low | medium | high | urgent (optional, default medium)",
  "planned_start_date": "date (optional)",
  "planned_end_date": "date (optional)"
}
```

**Response (201):** Task object

---

### GET /tasks/:id
Get task with dependencies.

**Response (200):**
```json
{
  "id": "uuid",
  "name": "string",
  "description": "string | null",
  "status": "string",
  "priority": "string",
  "planned_start_date": "date | null",
  "planned_end_date": "date | null",
  "actual_start_date": "date | null",
  "actual_end_date": "date | null",
  "phase": { "id": "uuid", "name": "string" } | null,
  "assigned_to": { "id": "uuid", "name": "string" } | null,
  "project": { "id": "uuid", "name": "string" },
  "dependencies": [
    {
      "id": "uuid",
      "task": { "id": "uuid", "name": "string", "status": "string" },
      "dependency_type": "finish_to_start | start_to_start | finish_to_finish | start_to_finish"
    }
  ],
  "dependents": [
    { "id": "uuid", "name": "string", "status": "string" }
  ],
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

---

### PUT /tasks/:id
Update task.

**Request:**
```json
{
  "name": "string (optional)",
  "description": "string (optional)",
  "phase_id": "uuid (optional)",
  "assigned_to": "uuid (optional)",
  "status": "string (optional)",
  "priority": "string (optional)",
  "planned_start_date": "date (optional)",
  "planned_end_date": "date (optional)",
  "actual_start_date": "date (optional)",
  "actual_end_date": "date (optional)"
}
```

**Response (200):** Updated task object

---

### DELETE /tasks/:id
Soft delete task.

**Response (200):**
```json
{
  "message": "Task deleted"
}
```

---

### POST /tasks/:id/dependencies
Add dependency.

**Request:**
```json
{
  "depends_on_task_id": "uuid (required)",
  "dependency_type": "finish_to_start (default) | start_to_start | finish_to_finish | start_to_finish"
}
```

**Response (201):**
```json
{
  "id": "uuid",
  "task_id": "uuid",
  "depends_on_task_id": "uuid",
  "dependency_type": "string"
}
```

**Errors:**
- 400: Self-dependency not allowed
- 400: Circular dependency detected
- 409: Dependency already exists

---

### DELETE /tasks/:id/dependencies/:depId
Remove dependency.

**Response (200):**
```json
{
  "message": "Dependency removed"
}
```

---

### GET /tasks/overdue
Get overdue tasks across all projects.

**Query Parameters:**
- `limit` (number, default 50)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "string",
      "planned_end_date": "date",
      "days_overdue": "number",
      "project": { "id": "uuid", "name": "string" },
      "assigned_to": { "id": "uuid", "name": "string" } | null
    }
  ],
  "total": "number"
}
```

---

### GET /projects/:id/timeline
Get timeline data for Gantt chart.

**Response (200):**
```json
{
  "phases": [
    {
      "id": "uuid",
      "name": "string",
      "start_date": "date | null",
      "end_date": "date | null",
      "status": "string",
      "tasks": [
        {
          "id": "uuid",
          "name": "string",
          "planned_start_date": "date | null",
          "planned_end_date": "date | null",
          "actual_start_date": "date | null",
          "actual_end_date": "date | null",
          "status": "string",
          "assigned_to": { "id": "uuid", "name": "string" } | null,
          "dependencies": [
            { "task_id": "uuid", "type": "string" }
          ]
        }
      ]
    }
  ],
  "milestones": [
    {
      "id": "uuid",
      "name": "string",
      "target_date": "date",
      "actual_date": "date | null",
      "status": "string"
    }
  ]
}
```

---

## Schedule - Milestones

### GET /projects/:id/milestones
List milestones.

**Query Parameters:**
- `phase_id` (uuid, optional)
- `status` (pending | achieved | missed, optional)
- `from_date` (date, optional)
- `to_date` (date, optional)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "string",
      "description": "string | null",
      "target_date": "date",
      "actual_date": "date | null",
      "status": "pending | achieved | missed",
      "phase": { "id": "uuid", "name": "string" } | null
    }
  ]
}
```

---

### POST /projects/:id/milestones
Create milestone.

**Request:**
```json
{
  "name": "string (required)",
  "target_date": "date (required)",
  "description": "string (optional)",
  "phase_id": "uuid (optional)"
}
```

**Response (201):** Milestone object

---

### PUT /milestones/:id
Update milestone.

**Request:**
```json
{
  "name": "string (optional)",
  "description": "string (optional)",
  "target_date": "date (optional)",
  "actual_date": "date (optional)",
  "status": "pending | achieved | missed (optional)",
  "phase_id": "uuid (optional)"
}
```

**Response (200):** Updated milestone object

---

### DELETE /milestones/:id
Soft delete milestone.

**Response (200):**
```json
{
  "message": "Milestone deleted"
}
```

---

### GET /milestones/upcoming
Get upcoming milestones across all projects.

**Query Parameters:**
- `days` (number, default 30)
- `limit` (number, default 20)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "string",
      "target_date": "date",
      "days_until": "number",
      "project": { "id": "uuid", "name": "string" }
    }
  ]
}
```

---

## Budget Categories

### GET /budget-categories
List budget categories.

**Query Parameters:**
- `is_active` (boolean, optional)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "string",
      "description": "string | null",
      "is_active": "boolean"
    }
  ]
}
```

---

### POST /budget-categories
Create category.

**Request:**
```json
{
  "name": "string (required)",
  "description": "string (optional)"
}
```

**Response (201):** Category object

**Errors:**
- 409: Category name already exists

---

### PUT /budget-categories/:id
Update category.

**Request:**
```json
{
  "name": "string (optional)",
  "description": "string (optional)",
  "is_active": "boolean (optional)"
}
```

**Response (200):** Updated category object

---

### DELETE /budget-categories/:id
Deactivate category.

**Response (200):**
```json
{
  "message": "Category deactivated"
}
```

**Errors:**
- 400: Category in use (has budget items)

---

## Budget Items

### GET /projects/:id/budget
Get full budget with items.

**Query Parameters:**
- `category_id` (uuid, optional)
- `phase_id` (uuid, optional)

**Response (200):**
```json
{
  "total_estimated": "number",
  "total_actual": "number",
  "variance": "number",
  "variance_percent": "number",
  "by_category": [
    {
      "category": { "id": "uuid", "name": "string" },
      "estimated": "number",
      "actual": "number",
      "variance": "number"
    }
  ],
  "items": [
    {
      "id": "uuid",
      "name": "string",
      "description": "string | null",
      "category": { "id": "uuid", "name": "string" },
      "phase": { "id": "uuid", "name": "string" } | null,
      "estimated_amount": "number",
      "actual_amount": "number"
    }
  ]
}
```

---

### GET /projects/:id/budget/summary
Get budget vs actual summary.

**Response (200):**
```json
{
  "total_estimated": "number",
  "total_actual": "number",
  "variance": "number",
  "variance_percent": "number",
  "cost_per_unit": "number",
  "by_category": [
    {
      "category_id": "uuid",
      "category_name": "string",
      "estimated": "number",
      "actual": "number",
      "variance": "number",
      "status": "on_track | over | under"
    }
  ]
}
```

---

### POST /projects/:id/budget-items
Create budget item.

**Request:**
```json
{
  "name": "string (required)",
  "category_id": "uuid (required)",
  "estimated_amount": "number (required)",
  "phase_id": "uuid (optional)",
  "description": "string (optional)"
}
```

**Response (201):** Budget item object

---

### PUT /budget-items/:id
Update budget item.

**Request:**
```json
{
  "name": "string (optional)",
  "category_id": "uuid (optional)",
  "phase_id": "uuid (optional)",
  "estimated_amount": "number (optional)",
  "actual_amount": "number (optional)",
  "description": "string (optional)"
}
```

**Response (200):** Updated budget item object

---

### DELETE /budget-items/:id
Soft delete budget item.

**Response (200):**
```json
{
  "message": "Budget item deleted"
}
```

---

### GET /projects/:id/spending-by-category
Get spending breakdown by category.

**Response (200):**
```json
{
  "data": [
    {
      "category_id": "uuid",
      "category_name": "string",
      "total_spent": "number",
      "percent_of_total": "number"
    }
  ],
  "total_spent": "number"
}
```

---

### GET /projects/:id/spending-by-phase
Get spending breakdown by phase.

**Response (200):**
```json
{
  "data": [
    {
      "phase_id": "uuid",
      "phase_name": "string",
      "total_spent": "number",
      "percent_of_total": "number"
    }
  ],
  "total_spent": "number"
}
```

---

## Documents

### GET /projects/:id/documents
List documents.

**Query Parameters:**
- `document_type` (invoice | estimate | contract | permit | photo | receipt | other, optional)
- `tags` (comma-separated strings, optional)
- `search` (string, searches name/description, optional)
- `sort` (created_at | name, default created_at)
- `order` (asc | desc, default desc)
- `limit` (number, default 20)
- `offset` (number, default 0)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "string",
      "description": "string | null",
      "document_type": "string",
      "file_size": "number",
      "mime_type": "string",
      "tags": ["string"],
      "uploaded_by": { "id": "uuid", "name": "string" },
      "created_at": "timestamp"
    }
  ],
  "total": "number",
  "limit": "number",
  "offset": "number"
}
```

---

### POST /projects/:id/documents
Upload document.

**Content-Type:** multipart/form-data

**Request:**
- `file` (file, required) - Max 50MB
- `document_type` (string, required)
- `description` (string, optional)
- `tags` (comma-separated strings, optional)

**Response (201):**
```json
{
  "id": "uuid",
  "name": "string",
  "document_type": "string",
  "file_size": "number",
  "mime_type": "string",
  "created_at": "timestamp"
}
```

**Errors:**
- 400: Invalid file type
- 400: File too large
- 507: Storage quota exceeded

---

### GET /documents/:id
Get document metadata.

**Response (200):**
```json
{
  "id": "uuid",
  "name": "string",
  "description": "string | null",
  "document_type": "string",
  "file_size": "number",
  "mime_type": "string",
  "tags": ["string"],
  "extracted_text": "string | null",
  "project": { "id": "uuid", "name": "string" },
  "uploaded_by": { "id": "uuid", "name": "string" },
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

---

### GET /documents/:id/download
Get signed download URL.

**Response (200):**
```json
{
  "url": "string - Signed R2 URL",
  "expires_at": "timestamp - 15 minutes from now"
}
```

---

### PUT /documents/:id
Update document metadata.

**Request:**
```json
{
  "name": "string (optional)",
  "description": "string (optional)",
  "document_type": "string (optional)"
}
```

**Response (200):** Updated document object

---

### DELETE /documents/:id
Soft delete document.

**Response (200):**
```json
{
  "message": "Document deleted"
}
```

---

### POST /documents/:id/tags
Add tags to document.

**Request:**
```json
{
  "tags": ["string", "string"]
}
```

**Response (200):**
```json
{
  "tags": ["string"]
}
```

---

### DELETE /documents/:id/tags/:tag
Remove tag from document.

**Response (200):**
```json
{
  "tags": ["string"]
}
```

---

### GET /documents/search
Search documents by content/tags.

**Query Parameters:**
- `q` (string, required) - Search query
- `project_id` (uuid, optional)
- `document_type` (string, optional)
- `limit` (number, default 20)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "string",
      "document_type": "string",
      "project": { "id": "uuid", "name": "string" },
      "snippet": "string - Matching text excerpt",
      "relevance_score": "number"
    }
  ],
  "total": "number"
}
```

---

## Invoices

### GET /projects/:id/invoices
List invoices.

**Query Parameters:**
- `status` (unpaid | partial | paid, optional)
- `category_id` (uuid, optional)
- `from_date` (date, optional)
- `to_date` (date, optional)
- `sort` (due_date | created_at | total_amount, default due_date)
- `order` (asc | desc, default asc)
- `limit` (number, default 20)
- `offset` (number, default 0)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "invoice_number": "string | null",
      "description": "string | null",
      "invoice_date": "date | null",
      "due_date": "date | null",
      "total_amount": "number",
      "paid_amount": "number",
      "status": "unpaid | partial | paid",
      "category": { "id": "uuid", "name": "string" } | null,
      "document": { "id": "uuid", "name": "string" } | null,
      "is_overdue": "boolean",
      "days_overdue": "number | null"
    }
  ],
  "total": "number",
  "summary": {
    "total_amount": "number",
    "paid_amount": "number",
    "outstanding": "number"
  }
}
```

---

### POST /projects/:id/invoices
Create invoice.

**Request:**
```json
{
  "invoice_number": "string (optional)",
  "description": "string (optional)",
  "total_amount": "number (required)",
  "invoice_date": "date (optional)",
  "due_date": "date (optional)",
  "category_id": "uuid (optional)",
  "document_id": "uuid (optional)",
  "notes": "string (optional)"
}
```

**Response (201):** Invoice object

---

### GET /invoices/:id
Get invoice with payments.

**Response (200):**
```json
{
  "id": "uuid",
  "invoice_number": "string | null",
  "description": "string | null",
  "invoice_date": "date | null",
  "due_date": "date | null",
  "total_amount": "number",
  "paid_amount": "number",
  "status": "string",
  "notes": "string | null",
  "category": { "id": "uuid", "name": "string" } | null,
  "document": { "id": "uuid", "name": "string" } | null,
  "project": { "id": "uuid", "name": "string" },
  "payments": [
    {
      "id": "uuid",
      "amount": "number",
      "payment_date": "date",
      "payment_method": "string | null",
      "reference_number": "string | null"
    }
  ],
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

---

### PUT /invoices/:id
Update invoice.

**Request:**
```json
{
  "invoice_number": "string (optional)",
  "description": "string (optional)",
  "total_amount": "number (optional)",
  "invoice_date": "date (optional)",
  "due_date": "date (optional)",
  "category_id": "uuid (optional)",
  "notes": "string (optional)"
}
```

**Response (200):** Updated invoice object

---

### DELETE /invoices/:id
Soft delete invoice.

**Response (200):**
```json
{
  "message": "Invoice deleted"
}
```

---

### GET /invoices/:id/payments
Get payments for invoice.

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "amount": "number",
      "payment_date": "date",
      "payment_method": "string | null",
      "reference_number": "string | null",
      "description": "string | null",
      "created_by": { "id": "uuid", "name": "string" },
      "created_at": "timestamp"
    }
  ],
  "total_paid": "number"
}
```

---

### GET /invoices/overdue
Get overdue invoices across all projects.

**Query Parameters:**
- `limit` (number, default 50)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "invoice_number": "string | null",
      "total_amount": "number",
      "paid_amount": "number",
      "outstanding": "number",
      "due_date": "date",
      "days_overdue": "number",
      "project": { "id": "uuid", "name": "string" }
    }
  ],
  "total": "number",
  "total_outstanding": "number"
}
```

---

## Payments

### GET /projects/:id/payments
List payments.

**Query Parameters:**
- `category_id` (uuid, optional)
- `from_date` (date, optional)
- `to_date` (date, optional)
- `payment_method` (check | credit_card | bank_transfer | cash | other, optional)
- `status` (pending | completed | cancelled, optional)
- `sort` (payment_date | amount, default payment_date)
- `order` (asc | desc, default desc)
- `limit` (number, default 20)
- `offset` (number, default 0)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "amount": "number",
      "payment_date": "date",
      "payment_method": "string | null",
      "reference_number": "string | null",
      "description": "string | null",
      "status": "string",
      "invoice": { "id": "uuid", "invoice_number": "string" } | null,
      "budget_item": { "id": "uuid", "name": "string", "category_name": "string" } | null,
      "category": { "id": "uuid", "name": "string" } | null,
      "created_by": { "id": "uuid", "name": "string" }
    }
  ],
  "total": "number",
  "summary": {
    "total_paid": "number",
    "outstanding": "number",
    "this_month": "number"
  }
}
```

---

### POST /projects/:id/payments
Record payment.

**Request:**
```json
{
  "amount": "number (required)",
  "payment_date": "date (required)",
  "description": "string (optional)",
  "payment_method": "check | credit_card | bank_transfer | cash | other (optional)",
  "reference_number": "string (optional)",
  "invoice_id": "uuid (optional)",
  "budget_item_id": "uuid (optional)",
  "category_id": "uuid (optional)"
}
```

**Response (201):** Payment object

**Side Effects:**
- If `invoice_id` provided, updates invoice `paid_amount` and `status`

---

### GET /payments/:id
Get payment details.

**Response (200):**
```json
{
  "id": "uuid",
  "amount": "number",
  "payment_date": "date",
  "payment_method": "string | null",
  "reference_number": "string | null",
  "description": "string | null",
  "status": "string",
  "invoice": { "id": "uuid", "invoice_number": "string" } | null,
  "budget_item": { "id": "uuid", "name": "string" } | null,
  "category": { "id": "uuid", "name": "string" } | null,
  "project": { "id": "uuid", "name": "string" },
  "created_by": { "id": "uuid", "name": "string" },
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

---

### PUT /payments/:id
Update payment.

**Request:**
```json
{
  "amount": "number (optional)",
  "payment_date": "date (optional)",
  "payment_method": "string (optional)",
  "reference_number": "string (optional)",
  "description": "string (optional)",
  "status": "string (optional)"
}
```

**Response (200):** Updated payment object

---

### DELETE /payments/:id
Soft delete payment.

**Response (200):**
```json
{
  "message": "Payment deleted"
}
```

---

## Task Items

### GET /task-items
List user's task items.

**Query Parameters:**
- `assigned_to` (me | uuid, optional, default me)
- `status` (pending | in_progress | completed | cancelled, optional)
- `priority` (low | medium | high | urgent, optional)
- `project_id` (uuid, optional)
- `due_before` (date, optional)
- `sort` (due_date | priority | created_at, default due_date)
- `order` (asc | desc, default asc)
- `limit` (number, default 20)
- `offset` (number, default 0)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "title": "string",
      "description": "string | null",
      "status": "string",
      "priority": "string",
      "due_date": "date | null",
      "is_overdue": "boolean",
      "project": { "id": "uuid", "name": "string" } | null,
      "assigned_to": { "id": "uuid", "name": "string" } | null,
      "created_by": { "id": "uuid", "name": "string" },
      "completed_at": "timestamp | null"
    }
  ],
  "total": "number",
  "counts": {
    "overdue": "number",
    "due_this_week": "number",
    "pending": "number",
    "completed": "number"
  }
}
```

---

### GET /projects/:id/task-items
List project task items.

**Query Parameters:** Same as GET /task-items

**Response:** Same format as GET /task-items

---

### POST /task-items
Create task item.

**Request:**
```json
{
  "title": "string (required)",
  "organization_id": "uuid (required)",
  "project_id": "uuid (optional)",
  "assigned_to": "uuid (optional)",
  "due_date": "date (optional)",
  "priority": "low | medium | high | urgent (optional, default medium)",
  "description": "string (optional)",
  "schedule_task_id": "uuid (optional) - Link to schedule task"
}
```

**Response (201):** Task item object

---

### GET /task-items/:id
Get task item details.

**Response (200):**
```json
{
  "id": "uuid",
  "title": "string",
  "description": "string | null",
  "status": "string",
  "priority": "string",
  "due_date": "date | null",
  "project": { "id": "uuid", "name": "string" } | null,
  "organization": { "id": "uuid", "name": "string" },
  "assigned_to": { "id": "uuid", "name": "string" } | null,
  "created_by": { "id": "uuid", "name": "string" },
  "completed_at": "timestamp | null",
  "completed_by": { "id": "uuid", "name": "string" } | null,
  "schedule_task": { "id": "uuid", "name": "string" } | null,
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

---

### PUT /task-items/:id
Update task item.

**Request:**
```json
{
  "title": "string (optional)",
  "description": "string (optional)",
  "status": "string (optional)",
  "priority": "string (optional)",
  "due_date": "date (optional)",
  "assigned_to": "uuid (optional)"
}
```

**Response (200):** Updated task item object

---

### PATCH /task-items/:id/complete
Mark task item as complete.

**Response (200):**
```json
{
  "id": "uuid",
  "status": "completed",
  "completed_at": "timestamp",
  "completed_by": { "id": "uuid", "name": "string" }
}
```

---

### DELETE /task-items/:id
Soft delete task item.

**Response (200):**
```json
{
  "message": "Task item deleted"
}
```

---

### GET /task-items/overdue
Get overdue task items.

**Query Parameters:**
- `limit` (number, default 50)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "title": "string",
      "due_date": "date",
      "days_overdue": "number",
      "priority": "string",
      "project": { "id": "uuid", "name": "string" } | null,
      "assigned_to": { "id": "uuid", "name": "string" } | null
    }
  ],
  "total": "number"
}
```

---

## Reminders

### GET /reminders
List user's reminders.

**Query Parameters:**
- `status` (pending | sent | cancelled, optional)
- `from_date` (datetime, optional)
- `to_date` (datetime, optional)
- `sort` (remind_at, default remind_at)
- `order` (asc | desc, default asc)
- `limit` (number, default 20)
- `offset` (number, default 0)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "title": "string",
      "description": "string | null",
      "remind_at": "timestamp",
      "recurrence": "none | daily | weekly | monthly",
      "status": "string",
      "related_type": "task_item | milestone | invoice | schedule_task | custom | null",
      "related_id": "uuid | null",
      "related_name": "string | null",
      "project": { "id": "uuid", "name": "string" } | null,
      "sent_at": "timestamp | null"
    }
  ],
  "total": "number"
}
```

---

### POST /reminders
Create reminder.

**Request:**
```json
{
  "title": "string (required)",
  "remind_at": "timestamp (required)",
  "description": "string (optional)",
  "recurrence": "none | daily | weekly | monthly (optional, default none)",
  "related_type": "task_item | milestone | invoice | schedule_task | custom (optional)",
  "related_id": "uuid (optional, required if related_type set except custom)",
  "project_id": "uuid (optional)",
  "organization_id": "uuid (optional)"
}
```

**Response (201):** Reminder object

---

### GET /reminders/:id
Get reminder details.

**Response (200):**
```json
{
  "id": "uuid",
  "title": "string",
  "description": "string | null",
  "remind_at": "timestamp",
  "recurrence": "string",
  "status": "string",
  "related_type": "string | null",
  "related_id": "uuid | null",
  "related": { ... } | null,
  "project": { "id": "uuid", "name": "string" } | null,
  "organization": { "id": "uuid", "name": "string" } | null,
  "sent_at": "timestamp | null",
  "notification_id": "uuid | null",
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

---

### PUT /reminders/:id
Update reminder.

**Request:**
```json
{
  "title": "string (optional)",
  "description": "string (optional)",
  "remind_at": "timestamp (optional)",
  "recurrence": "string (optional)"
}
```

**Response (200):** Updated reminder object

---

### DELETE /reminders/:id
Cancel/delete reminder.

**Response (200):**
```json
{
  "message": "Reminder cancelled"
}
```

---

### PATCH /reminders/:id/cancel
Cancel reminder (sets status to cancelled).

**Response (200):**
```json
{
  "id": "uuid",
  "status": "cancelled"
}
```

---

## Notifications

### GET /notifications
List notifications.

**Query Parameters:**
- `read` (boolean, optional) - Filter by read status
- `notification_type` (string, optional)
- `limit` (number, default 50)
- `offset` (number, default 0)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "notification_type": "reminder | task_assigned | task_due | milestone_approaching | invoice_due | budget_alert | system",
      "title": "string",
      "body": "string | null",
      "action_url": "string | null",
      "data": "object",
      "priority": "low | normal | high",
      "read_at": "timestamp | null",
      "created_at": "timestamp"
    }
  ],
  "total": "number",
  "unread_count": "number"
}
```

---

### GET /notifications/unread-count
Get unread notification count.

**Response (200):**
```json
{
  "count": "number"
}
```

---

### PATCH /notifications/:id/read
Mark notification as read.

**Response (200):**
```json
{
  "id": "uuid",
  "read_at": "timestamp"
}
```

---

### POST /notifications/read-all
Mark all notifications as read.

**Response (200):**
```json
{
  "updated_count": "number"
}
```

---

### DELETE /notifications/:id
Delete notification.

**Response (200):**
```json
{
  "message": "Notification deleted"
}
```

---

## Alert Thresholds

### GET /alert-thresholds
List alert configurations.

**Query Parameters:**
- `project_id` (uuid, optional)
- `alert_type` (budget_percent | budget_amount | schedule_variance_days, optional)
- `is_active` (boolean, optional)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "alert_type": "string",
      "threshold_value": "number",
      "is_active": "boolean",
      "project": { "id": "uuid", "name": "string" } | null,
      "last_triggered_at": "timestamp | null"
    }
  ]
}
```

---

### POST /alert-thresholds
Create alert threshold.

**Request:**
```json
{
  "alert_type": "budget_percent | budget_amount | schedule_variance_days (required)",
  "threshold_value": "number (required)",
  "project_id": "uuid (optional) - If null, applies to all projects"
}
```

**Response (201):** Alert threshold object

---

### PUT /alert-thresholds/:id
Update threshold.

**Request:**
```json
{
  "threshold_value": "number (optional)",
  "is_active": "boolean (optional)"
}
```

**Response (200):** Updated threshold object

---

### DELETE /alert-thresholds/:id
Delete threshold.

**Response (200):**
```json
{
  "message": "Alert threshold deleted"
}
```

---

## Activity Feed

### GET /activity
Get activity feed.

**Query Parameters:**
- `organization_id` (uuid, optional)
- `project_id` (uuid, optional)
- `user_id` (uuid, optional)
- `entity_type` (project | task | document | payment | invoice, optional)
- `limit` (number, default 50)
- `offset` (number, default 0)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "action": "created | updated | deleted | uploaded | assigned | completed",
      "entity_type": "string",
      "entity_id": "uuid",
      "entity_name": "string | null",
      "user": { "id": "uuid", "name": "string" } | null,
      "project": { "id": "uuid", "name": "string" } | null,
      "changes": "object | null",
      "created_at": "timestamp"
    }
  ],
  "total": "number"
}
```

---

## Global Search

### GET /search
Search across all entities.

**Query Parameters:**
- `q` (string, required) - Search query
- `type` (all | projects | documents | tasks | invoices, optional, default all)
- `limit` (number, default 5 per type)

**Response (200):**
```json
{
  "projects": [
    { "id": "uuid", "name": "string", "address": "string", "units": "number", "status": "string" }
  ],
  "documents": [
    { "id": "uuid", "name": "string", "project_name": "string", "document_type": "string" }
  ],
  "tasks": [
    { "id": "uuid", "title": "string", "project_name": "string", "status": "string" }
  ],
  "invoices": [
    { "id": "uuid", "invoice_number": "string", "amount": "number", "status": "string" }
  ]
}
```

---

## Dashboard

### GET /dashboard/summary
Get dashboard summary statistics.

**Response (200):**
```json
{
  "active_projects": "number",
  "total_units": "number",
  "total_budget": "number",
  "total_spent": "number",
  "overdue_tasks": "number",
  "unpaid_invoices_amount": "number",
  "upcoming_milestones": "number"
}
```

---

## AI Conversations

### GET /ai/conversations
List conversations.

**Query Parameters:**
- `limit` (number, default 20)
- `offset` (number, default 0)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "title": "string | null",
      "project": { "id": "uuid", "name": "string" } | null,
      "message_count": "number",
      "created_at": "timestamp",
      "updated_at": "timestamp"
    }
  ],
  "total": "number"
}
```

---

### POST /ai/conversations
Start new conversation.

**Request:**
```json
{
  "project_id": "uuid (optional) - Scope to specific project"
}
```

**Response (201):**
```json
{
  "id": "uuid",
  "title": null,
  "project": { "id": "uuid", "name": "string" } | null,
  "created_at": "timestamp"
}
```

---

### GET /ai/conversations/:id
Get conversation with messages.

**Response (200):**
```json
{
  "id": "uuid",
  "title": "string | null",
  "project": { "id": "uuid", "name": "string" } | null,
  "messages": [
    {
      "id": "uuid",
      "role": "user | assistant",
      "content": "string",
      "query_type": "spending | schedule | budget | comparison | trend | document | open_ended | null",
      "sources": [
        { "type": "string", "id": "uuid", "name": "string" }
      ] | null,
      "tokens_used": "number | null",
      "created_at": "timestamp"
    }
  ],
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

---

### DELETE /ai/conversations/:id
Delete conversation.

**Response (200):**
```json
{
  "message": "Conversation deleted"
}
```

---

### POST /ai/conversations/:id/messages
Send message and get response.

**Request:**
```json
{
  "message": "string (required)",
  "context": {
    "projectId": "uuid (optional) - Override conversation's project scope"
  }
}
```

**Response (200):**
```json
{
  "id": "uuid",
  "role": "assistant",
  "content": "string",
  "query_type": "string | null",
  "sources": [
    { "type": "string", "id": "uuid", "name": "string" }
  ] | null,
  "tokens_used": "number",
  "created_at": "timestamp"
}
```

**Errors:**
- 429: Monthly AI token limit exceeded

---

### POST /ai/feedback
Submit feedback on AI response.

**Request:**
```json
{
  "message_id": "uuid (required)",
  "rating": "number (required) - 1 (thumbs down) or 5 (thumbs up)",
  "feedback_text": "string (optional)"
}
```

**Response (201):**
```json
{
  "id": "uuid",
  "message": "Feedback recorded"
}
```

---

### GET /ai/insights
Get active AI insights.

**Query Parameters:**
- `project_id` (uuid, optional)
- `insight_type` (cost_pattern | schedule_risk | efficiency | anomaly | recommendation, optional)
- `is_dismissed` (boolean, optional, default false)
- `limit` (number, default 20)

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "insight_type": "string",
      "title": "string",
      "description": "string",
      "severity": "info | warning | critical",
      "data": "object | null",
      "project": { "id": "uuid", "name": "string" } | null,
      "created_at": "timestamp"
    }
  ]
}
```

---

### POST /ai/insights/:id/dismiss
Dismiss insight.

**Response (200):**
```json
{
  "id": "uuid",
  "is_dismissed": true,
  "dismissed_at": "timestamp"
}
```

---

### GET /ai/suggestions
Get contextual query suggestions.

**Query Parameters:**
- `project_id` (uuid, optional)

**Response (200):**
```json
{
  "suggestions": [
    "What tasks are overdue?",
    "How's the budget looking?",
    "What's my cost per unit?"
  ]
}
```

---

## Common Response Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request - Validation error |
| 401 | Unauthorized - Invalid/missing token |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found |
| 409 | Conflict - Resource already exists |
| 422 | Unprocessable Entity |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error |
| 507 | Insufficient Storage - Quota exceeded |

---

## Error Response Format

All errors return:

```json
{
  "error": {
    "code": "string - Machine-readable error code",
    "message": "string - Human-readable message",
    "details": "object | null - Additional context"
  }
}
```

Example:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request body",
    "details": {
      "fields": {
        "email": "Must be a valid email address",
        "password": "Must be at least 8 characters"
      }
    }
  }
}
```

---

## Rate Limits

| Endpoint Type | Limit |
|---------------|-------|
| General API | 100 requests/minute per user |
| Auth endpoints | 10 requests/minute per IP |
| File upload | 10 requests/minute per user |
| AI queries | 20 requests/minute per user |
| Builder total | 1000 requests/minute |

Rate limit headers included in responses:
- `X-RateLimit-Limit`
- `X-RateLimit-Remaining`
- `X-RateLimit-Reset`

---

## Calculated Fields

Fields that appear in API responses but are computed at query time (not stored in database):

| Field | Formula | Used In |
|-------|---------|---------|
| `costPerUnit` | `totalSpent / units` | Project summary, comparison endpoints |
| `budgetVariance` | `totalEstimated - totalActual` | Budget summary endpoints |
| `budgetVariancePercent` | `(variance / totalEstimated) * 100` | Budget summary endpoints |
| `scheduleProgress` | `completedTasks / totalTasks` | Project overview, timeline endpoints |
| `daysRemaining` | `endDate - today` | Project summary, milestones |
| `daysOverdue` | `today - dueDate` (when overdue) | Tasks, milestones, invoices |

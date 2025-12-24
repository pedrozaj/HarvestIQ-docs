# HarvestIQ API Documentation

## Backend API

**Base URL:** `https://harvestiq-backend-production.up.railway.app`

---

## Health Check

```
GET /health
```

**Response:**
```json
{
  "status": "ok",
  "timestamp": "2025-12-23T21:00:00.000Z"
}
```

```
GET /
```

**Response:**
```json
{
  "name": "HarvestIQ API",
  "version": "1.0.0",
  "environment": "production"
}
```

---

## Authentication

All auth endpoints are prefixed with `/api/auth`.

### Register

Create a new builder account with initial user and organization.

```
POST /api/auth/register
Content-Type: application/json
```

**Rate Limit:** 5 requests/minute per IP

**Request Body:**
```json
{
  "company_name": "Acme Builders",
  "name": "John Smith",
  "email": "john@acmebuilders.com",
  "phone": "(555) 123-4567",
  "password": "SecurePass123"
}
```

**Response (201 Created):**
```json
{
  "data": {
    "message": "Verification email sent"
  }
}
```

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| AUTH_1005 | 409 | Email already registered |
| AUTH_1006 | 400 | Password does not meet requirements |
| VAL_3001 | 400 | Validation failed |

---

### Verify Email

Verify email address with token from verification email.

```
POST /api/auth/verify-email
Content-Type: application/json
```

**Request Body:**
```json
{
  "token": "abc123..."
}
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Email verified successfully"
  }
}
```

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| AUTH_1003 | 400 | Invalid or expired verification token |

---

### Resend Verification

Resend verification email.

```
POST /api/auth/resend-verification
Content-Type: application/json
```

**Rate Limit:** 5 requests/minute per IP

**Request Body:**
```json
{
  "email": "john@acmebuilders.com"
}
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "If account exists and is unverified, verification email sent"
  }
}
```

---

### Login

Authenticate user and receive tokens.

```
POST /api/auth/login
Content-Type: application/json
```

**Rate Limit:** 10 requests/minute per IP

**Request Body:**
```json
{
  "email": "john@acmebuilders.com",
  "password": "SecurePass123"
}
```

**Response (200 OK):**
```json
{
  "data": {
    "user": {
      "id": "uuid",
      "email": "john@acmebuilders.com",
      "name": "John Smith",
      "builderId": "uuid"
    },
    "accessToken": "eyJhbG..."
  }
}
```

**Cookies Set:**
- `access_token` - JWT access token (15 min expiry)
- `refresh_token` - JWT refresh token (7 day expiry)

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| AUTH_1001 | 401 | Invalid email or password |
| AUTH_1007 | 403 | Email not verified |
| AUTH_1008 | 423 | Account locked. Try again in X minutes |

---

### Logout

Invalidate current session and revoke refresh token.

```
POST /api/auth/logout
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Logged out successfully"
  }
}
```

---

### Refresh Token

Get new access token using refresh token.

```
POST /api/auth/refresh
```

**Rate Limit:** 30 requests/minute per user

**Response (200 OK):**
```json
{
  "data": {
    "accessToken": "eyJhbG..."
  }
}
```

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| AUTH_1004 | 401 | Refresh token expired or invalid |
| AUTH_1009 | 403 | Subscription inactive |

---

### Forgot Password

Initiate password reset flow.

```
POST /api/auth/forgot-password
Content-Type: application/json
```

**Rate Limit:** 3 requests/minute per email

**Request Body:**
```json
{
  "email": "john@acmebuilders.com"
}
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "If an account exists, a reset email has been sent"
  }
}
```

---

### Reset Password

Complete password reset with token.

```
POST /api/auth/reset-password
Content-Type: application/json
```

**Request Body:**
```json
{
  "token": "abc123...",
  "password": "NewSecurePass123"
}
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Password reset successfully"
  }
}
```

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| AUTH_1003 | 400 | Invalid or expired reset token |
| AUTH_1006 | 400 | Password does not meet requirements |

---

## User Endpoints

All user endpoints require authentication and are prefixed with `/api/users`.

### Get Current User

```
GET /api/users/me
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "email": "john@acmebuilders.com",
    "name": "John Smith",
    "phone": "(555) 123-4567",
    "timezone": "America/New_York",
    "emailVerifiedAt": "2025-12-23T21:00:00.000Z",
    "lastLoginAt": "2025-12-23T21:00:00.000Z",
    "builder": {
      "id": "uuid",
      "name": "Acme Builders",
      "subscriptionPlan": "trial",
      "subscriptionStatus": "trial"
    },
    "organizations": [
      {
        "id": "uuid",
        "name": "Acme Builders",
        "role": "admin"
      }
    ],
    "createdAt": "2025-12-23T21:00:00.000Z"
  }
}
```

---

### Update Current User

```
PUT /api/users/me
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "John Smith Jr.",
  "phone": "(555) 987-6543",
  "timezone": "America/Los_Angeles"
}
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "email": "john@acmebuilders.com",
    "name": "John Smith Jr.",
    "phone": "(555) 987-6543",
    "timezone": "America/Los_Angeles",
    "updatedAt": "2025-12-23T21:00:00.000Z"
  }
}
```

---

### Change Password

```
PUT /api/users/me/password
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "currentPassword": "OldSecurePass123",
  "newPassword": "NewSecurePass456"
}
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Password changed successfully"
  }
}
```

**Side Effects:** Revokes all other sessions.

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| AUTH_1001 | 400 | Current password is incorrect |
| AUTH_1006 | 400 | New password does not meet requirements |

---

### Update Notification Preferences

```
PUT /api/users/me/notification-preferences
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "channels": {
    "email": true,
    "sms": false,
    "in_app": true
  },
  "types": {
    "reminder": ["email", "in_app"],
    "task_assigned": ["in_app"],
    "task_due": ["email", "in_app"],
    "milestone_approaching": ["email", "in_app"],
    "invoice_due": ["email"],
    "system": ["in_app"]
  }
}
```

**Response (200 OK):**
```json
{
  "data": {
    "channels": { ... },
    "types": { ... }
  }
}
```

---

### List Active Sessions

```
GET /api/users/me/sessions
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "uuid",
      "deviceInfo": "Mozilla/5.0...",
      "createdAt": "2025-12-23T21:00:00.000Z",
      "isCurrent": true
    }
  ]
}
```

---

### Revoke Session

```
DELETE /api/users/me/sessions/:id
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Session revoked"
  }
}
```

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| RES_4001 | 404 | Session not found |
| AUTHZ_2002 | 403 | Cannot revoke current session |

---

### Revoke All Other Sessions

```
DELETE /api/users/me/sessions
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "revokedCount": 3
  }
}
```

---

## Builder Endpoints

All builder endpoints require authentication and are prefixed with `/api/builder`.

### Get Builder

```
GET /api/builder
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "name": "Acme Builders",
    "subscriptionStatus": "trial",
    "subscriptionPlan": "free",
    "trialEndsAt": "2025-01-06T21:00:00.000Z",
    "storageUsedBytes": 0,
    "storageLimitBytes": 5368709120,
    "aiTokensUsedMonth": 0,
    "aiTokensLimitMonth": 100000,
    "createdAt": "2025-12-23T21:00:00.000Z"
  }
}
```

---

### Update Builder

```
PUT /api/builder
Authorization: Bearer <token>
Content-Type: application/json
```

**Requires:** Admin role

**Request Body:**
```json
{
  "name": "Acme Builders Inc.",
  "billingEmail": "billing@acmebuilders.com"
}
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "name": "Acme Builders Inc.",
    ...
  }
}
```

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| AUTHZ_2003 | 403 | Admin role required |

---

## Error Response Format

All errors return:

```json
{
  "error": {
    "code": "AUTH_1001",
    "message": "Human-readable message",
    "details": {}
  }
}
```

**Error Code Prefixes:**
- `AUTH_1xxx` - Authentication errors
- `AUTHZ_2xxx` - Authorization errors
- `VAL_3xxx` - Validation errors
- `RES_4xxx` - Resource errors
- `SYS_9xxx` - System errors

---

## Authentication

Requests are authenticated using JWT tokens:

**Cookie-based (preferred for web):**
```
Cookie: access_token=<jwt_token>
```

**Header-based:**
```
Authorization: Bearer <jwt_token>
```

---

## Rate Limiting

| Endpoint | Limit | Key |
|----------|-------|-----|
| POST /api/auth/login | 10/minute | IP |
| POST /api/auth/register | 5/minute | IP |
| POST /api/auth/forgot-password | 3/minute | email |
| POST /api/auth/refresh | 30/minute | user |
| All other endpoints | 100/minute | user |

Rate limit headers:
- `X-RateLimit-Limit`
- `X-RateLimit-Remaining`
- `X-RateLimit-Reset`

---

## CORS

The API restricts CORS to the frontend domain in production:

```
Access-Control-Allow-Origin: https://harvest-iq-rosy.vercel.app
Access-Control-Allow-Credentials: true
```

---

## Organization Endpoints

All organization endpoints require authentication and are prefixed with `/api/organizations`.

### List Organizations

```
GET /api/organizations
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "Acme Builders",
      "isDefault": true,
      "settings": {},
      "createdAt": "2025-12-23T21:00:00.000Z",
      "updatedAt": "2025-12-23T21:00:00.000Z"
    }
  ]
}
```

---

### Get Organization

```
GET /api/organizations/:id
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "name": "Acme Builders",
    "isDefault": true,
    "settings": {},
    "memberCount": 5,
    "projectCount": 12,
    "createdAt": "2025-12-23T21:00:00.000Z",
    "updatedAt": "2025-12-23T21:00:00.000Z"
  }
}
```

---

### Create Organization

```
POST /api/organizations
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "New Division"
}
```

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "name": "New Division",
    "isDefault": false,
    "settings": {},
    "createdAt": "2025-12-23T21:00:00.000Z",
    "updatedAt": "2025-12-23T21:00:00.000Z"
  }
}
```

---

### Update Organization

```
PATCH /api/organizations/:id
Authorization: Bearer <token>
Content-Type: application/json
```

**Requires:** Admin role in the organization

**Request Body:**
```json
{
  "name": "Updated Name",
  "settings": {}
}
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "name": "Updated Name",
    ...
  }
}
```

---

### Delete Organization

```
DELETE /api/organizations/:id
Authorization: Bearer <token>
```

**Requires:** Admin role in the organization

**Response (200 OK):**
```json
{
  "data": {
    "message": "Organization deleted"
  }
}
```

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| RES_4003 | 400 | Cannot delete default organization |
| AUTHZ_2002 | 403 | Only admin can delete organization |

---

### List Organization Members

```
GET /api/organizations/:id/members
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "uuid",
      "userId": "uuid",
      "email": "user@example.com",
      "name": "John Smith",
      "role": "admin",
      "joinedAt": "2025-12-23T21:00:00.000Z",
      "isActive": true
    }
  ]
}
```

---

### Update Member Role

```
PATCH /api/organizations/:id/members/:userId
Authorization: Bearer <token>
Content-Type: application/json
```

**Requires:** Admin role in the organization

**Request Body:**
```json
{
  "role": "manager"
}
```

**Role Values:** `admin`, `manager`, `member`, `viewer`

**Response (200 OK):**
```json
{
  "data": {
    "message": "Role updated"
  }
}
```

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| BIZ_5001 | 400 | Cannot demote last admin |
| AUTHZ_2002 | 403 | Only admin can change member roles |

---

### Remove Member

```
DELETE /api/organizations/:id/members/:userId
Authorization: Bearer <token>
```

**Requires:** Admin role in the organization

**Response (200 OK):**
```json
{
  "data": {
    "message": "Member removed"
  }
}
```

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| BIZ_5002 | 400 | Cannot remove yourself from organization |
| AUTHZ_2002 | 403 | Only admin can remove members |

---

### List Pending Invitations

```
GET /api/organizations/:organizationId/invitations
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "uuid",
      "email": "new@example.com",
      "role": "member",
      "invitedBy": "uuid",
      "message": "Join our team!",
      "expiresAt": "2025-12-30T21:00:00.000Z",
      "createdAt": "2025-12-23T21:00:00.000Z"
    }
  ]
}
```

---

### Create Invitation

```
POST /api/organizations/:organizationId/invitations
Authorization: Bearer <token>
Content-Type: application/json
```

**Requires:** Admin role in the organization

**Request Body:**
```json
{
  "organizationId": "uuid",
  "email": "new@example.com",
  "role": "member",
  "message": "Join our team!"
}
```

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "email": "new@example.com",
    "role": "member",
    "invitedBy": "uuid",
    "message": "Join our team!",
    "expiresAt": "2025-12-30T21:00:00.000Z",
    "createdAt": "2025-12-23T21:00:00.000Z"
  }
}
```

**Side Effects:** Sends invitation email to the recipient.

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| RES_4002 | 409 | User is already a member of this organization |
| RES_4002 | 409 | Invitation already pending for this email |
| AUTHZ_2002 | 403 | Only admin can invite members |

---

## Invitation Endpoints

Invitation management endpoints are prefixed with `/api/invitations`.

### Get Invitation Info

Get invitation details by token (public endpoint).

```
GET /api/invitations/:token
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "email": "new@example.com",
    "role": "member",
    "organizationName": "Acme Builders",
    "inviterName": "John Smith",
    "expiresAt": "2025-12-30T21:00:00.000Z"
  }
}
```

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| BIZ_5003 | 400 | Invalid or expired invitation |

---

### Accept Invitation

```
POST /api/invitations/:token/accept
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Invitation accepted",
    "organizationId": "uuid",
    "organizationName": "Acme Builders"
  }
}
```

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| BIZ_5003 | 400 | Invalid or expired invitation |
| AUTHZ_2001 | 403 | Invitation was sent to a different email |
| BIZ_5004 | 400 | Invitation is for a different account |

---

### Resend Invitation

```
POST /api/invitations/:id/resend
Authorization: Bearer <token>
```

**Requires:** Admin role in the organization

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "email": "new@example.com",
    "role": "member",
    ...
  }
}
```

**Side Effects:** Sends new invitation email and resets expiration.

---

### Cancel Invitation

```
DELETE /api/invitations/:id
Authorization: Bearer <token>
```

**Requires:** Admin role in the organization

**Response (200 OK):**
```json
{
  "data": {
    "message": "Invitation cancelled"
  }
}
```

---

## Project Endpoints

All project endpoints require authentication and are prefixed with `/api/projects`.

### List Projects

```
GET /api/projects
Authorization: Bearer <token>
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| status | string | Filter by status: `planning`, `active`, `on_hold`, `completed` |
| organizationId | uuid | Filter by organization |
| unitType | string | Filter by unit type: `single_family`, `townhomes`, `condos`, `apartments` |
| search | string | Search in name and address |
| sortBy | string | Sort field: `created_at`, `name`, `units`, `status`, `start_date` |
| sortOrder | string | Sort direction: `asc`, `desc` |
| limit | number | Results per page (default: 20, max: 100) |
| offset | number | Pagination offset |

**Response (200 OK):**
```json
{
  "data": {
    "projects": [
      {
        "id": "uuid",
        "organizationId": "uuid",
        "name": "Riverside Apartments",
        "address": "123 Main St",
        "unitType": "apartments",
        "units": 24,
        "lotSizeAcres": 2.5,
        "status": "active",
        "startDate": "2025-01-15",
        "endDate": "2025-12-31",
        "description": "24-unit apartment complex",
        "createdAt": "2025-12-23T21:00:00.000Z",
        "updatedAt": "2025-12-23T21:00:00.000Z"
      }
    ],
    "total": 1,
    "limit": 20,
    "offset": 0
  }
}
```

---

### Create Project

```
POST /api/projects
Authorization: Bearer <token>
Content-Type: application/json
```

**Requires:** Admin or Manager role in the organization

**Request Body:**
```json
{
  "organizationId": "uuid",
  "name": "Riverside Apartments",
  "address": "123 Main St",
  "unitType": "apartments",
  "units": 24,
  "lotSizeAcres": 2.5,
  "startDate": "2025-01-15",
  "endDate": "2025-12-31",
  "description": "24-unit apartment complex"
}
```

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "organizationId": "uuid",
    "name": "Riverside Apartments",
    ...
  }
}
```

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| RES_4001 | 404 | Organization not found |
| AUTHZ_2001 | 403 | Not a member of this organization |
| AUTHZ_2002 | 403 | Only admin or manager can create projects |

---

### Get Project

```
GET /api/projects/:id
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "organizationId": "uuid",
    "name": "Riverside Apartments",
    ...
  }
}
```

---

### Update Project

```
PATCH /api/projects/:id
Authorization: Bearer <token>
Content-Type: application/json
```

**Requires:** Admin or Manager role in the organization

**Request Body:**
```json
{
  "name": "Updated Name",
  "status": "on_hold",
  ...
}
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "name": "Updated Name",
    "status": "on_hold",
    ...
  }
}
```

---

### Delete Project

```
DELETE /api/projects/:id
Authorization: Bearer <token>
```

**Requires:** Admin role in the organization

**Response (200 OK):**
```json
{
  "data": {
    "message": "Project deleted"
  }
}
```

---

### Get Project Summary

Get aggregated project statistics including budget, schedule, and milestones.

```
GET /api/projects/:id/summary
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "name": "Riverside Apartments",
    "status": "active",
    "units": 24,
    "budget": {
      "totalEstimated": 500000,
      "totalActual": 125000,
      "variance": 375000,
      "variancePercent": 75
    },
    "schedule": {
      "totalTasks": 50,
      "completedTasks": 12,
      "overdueTasks": 2,
      "progressPercent": 24
    },
    "milestones": {
      "total": 8,
      "achieved": 2,
      "upcoming": 6,
      "nextMilestone": {
        "name": "Foundation Complete",
        "targetDate": "2025-02-15"
      }
    },
    "payments": {
      "totalPaid": 0,
      "outstandingInvoices": 0,
      "outstandingAmount": 0
    },
    "costPerUnit": 5208.33,
    "daysRemaining": 180
  }
}
```

---

### Compare Projects

Compare multiple projects side by side.

```
GET /api/projects/compare?ids=uuid1,uuid2,uuid3
Authorization: Bearer <token>
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| ids | string | Comma-separated list of 2-5 project UUIDs |

**Response (200 OK):**
```json
{
  "data": {
    "projects": [
      {
        "id": "uuid",
        "name": "Riverside Apartments",
        "status": "active",
        "units": 24,
        "unitType": "apartments",
        "budget": {
          "totalEstimated": 500000,
          "totalActual": 125000,
          "variance": 375000,
          "variancePercent": 75
        },
        "schedule": {
          "totalTasks": 50,
          "completedTasks": 12,
          "overdueTasks": 2,
          "progressPercent": 24
        },
        "milestones": {
          "total": 8,
          "achieved": 2,
          "upcoming": 6
        },
        "costPerUnit": 5208.33,
        "daysRemaining": 180
      }
    ],
    "count": 2
  }
}
```

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| VAL_3001 | 400 | Project IDs required (comma-separated) |
| VAL_3001 | 400 | Provide 2-5 project IDs to compare |

---

### Get Project Activity

```
GET /api/projects/:id/activity
Authorization: Bearer <token>
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| limit | number | Results per page (default: 50, max: 100) |
| offset | number | Pagination offset |

**Response (200 OK):**
```json
{
  "data": {
    "activities": [
      {
        "id": "uuid",
        "userId": "uuid",
        "userName": "John Smith",
        "entityType": "project",
        "entityId": "uuid",
        "action": "created",
        "changes": null,
        "metadata": { "name": "Riverside Apartments" },
        "createdAt": "2025-12-23T21:00:00.000Z"
      }
    ],
    "total": 1
  }
}
```

---

## Activity Endpoints

Activity log endpoints are prefixed with `/api/activity`.

### List Activity

```
GET /api/activity
Authorization: Bearer <token>
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| projectId | uuid | Filter by project |
| entityType | string | Filter by entity: `project`, `task`, `budget_item`, `document`, `payment`, `invoice`, `organization`, `user` |
| limit | number | Results per page (default: 50, max: 100) |
| offset | number | Pagination offset |

**Response (200 OK):**
```json
{
  "data": {
    "activities": [
      {
        "id": "uuid",
        "userId": "uuid",
        "userName": "John Smith",
        "projectId": "uuid",
        "projectName": "Riverside Apartments",
        "entityType": "project",
        "entityId": "uuid",
        "action": "status_changed",
        "changes": {
          "status": { "from": "planning", "to": "active" }
        },
        "metadata": null,
        "createdAt": "2025-12-23T21:00:00.000Z"
      }
    ],
    "total": 25,
    "limit": 50,
    "offset": 0
  }
}
```

**Action Values:** `created`, `updated`, `deleted`, `status_changed`, `assigned`, `completed`, `uploaded`, `payment_recorded`

---

## Schedule Phase Endpoints

Schedule phases are nested under projects: `/api/projects/:id/phases`.

### List Phases

```
GET /api/projects/:id/phases
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "uuid",
      "projectId": "uuid",
      "name": "Foundation",
      "description": "Foundation work",
      "status": "not_started",
      "sortOrder": 1,
      "startDate": "2025-01-15",
      "endDate": "2025-02-15",
      "createdAt": "2025-12-23T21:00:00.000Z",
      "updatedAt": "2025-12-23T21:00:00.000Z"
    }
  ]
}
```

---

### Create Phase

```
POST /api/projects/:id/phases
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "Foundation",
  "description": "Foundation work",
  "startDate": "2025-01-15",
  "endDate": "2025-02-15"
}
```

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "projectId": "uuid",
    "name": "Foundation",
    ...
  }
}
```

---

### Update Phase

```
PUT /api/projects/:id/phases/:phaseId
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "Updated Name",
  "status": "in_progress",
  "startDate": "2025-01-15",
  "endDate": "2025-02-28"
}
```

**Status Values:** `not_started`, `in_progress`, `completed`

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "name": "Updated Name",
    "status": "in_progress",
    ...
  }
}
```

---

### Delete Phase

```
DELETE /api/projects/:id/phases/:phaseId
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Phase deleted"
  }
}
```

---

### Reorder Phases

```
PUT /api/projects/:id/phases/reorder
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "phases": [
    { "id": "uuid-1", "sortOrder": 0 },
    { "id": "uuid-2", "sortOrder": 1 },
    { "id": "uuid-3", "sortOrder": 2 }
  ]
}
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Phases reordered"
  }
}
```

---

## Schedule Task Endpoints

Schedule tasks are nested under projects: `/api/projects/:id/tasks`.

### List Tasks

```
GET /api/projects/:id/tasks
Authorization: Bearer <token>
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| phaseId | uuid | Filter by phase |
| status | string | Filter by status: `not_started`, `in_progress`, `completed`, `blocked` |
| assignedTo | uuid | Filter by assigned user |
| priority | string | Filter by priority: `low`, `medium`, `high`, `urgent` |
| limit | number | Results per page (default: 20, max: 100) |
| offset | number | Pagination offset |

**Response (200 OK):**
```json
{
  "data": {
    "tasks": [
      {
        "id": "uuid",
        "projectId": "uuid",
        "phaseId": "uuid",
        "name": "Pour footings",
        "description": "Pour concrete footings",
        "status": "not_started",
        "priority": "high",
        "assignedTo": "uuid",
        "assigneeName": "John Smith",
        "plannedStartDate": "2025-01-20",
        "plannedEndDate": "2025-01-22",
        "actualStartDate": null,
        "actualEndDate": null,
        "estimatedHours": 16,
        "actualHours": null,
        "createdAt": "2025-12-23T21:00:00.000Z",
        "updatedAt": "2025-12-23T21:00:00.000Z"
      }
    ],
    "total": 1
  }
}
```

---

### Create Task

```
POST /api/projects/:id/tasks
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "phaseId": "uuid",
  "name": "Pour footings",
  "description": "Pour concrete footings",
  "assignedTo": "uuid",
  "priority": "high",
  "plannedStartDate": "2025-01-20",
  "plannedEndDate": "2025-01-22",
  "estimatedHours": 16
}
```

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "projectId": "uuid",
    "phaseId": "uuid",
    "name": "Pour footings",
    ...
  }
}
```

---

### Get Task

```
GET /api/projects/:id/tasks/:taskId
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "projectId": "uuid",
    "phaseId": "uuid",
    "name": "Pour footings",
    "description": "Pour concrete footings",
    "status": "not_started",
    "priority": "high",
    "dependencies": [
      {
        "id": "uuid",
        "dependsOnTaskId": "uuid",
        "dependsOnTaskName": "Excavate foundation"
      }
    ],
    ...
  }
}
```

---

### Update Task

```
PUT /api/projects/:id/tasks/:taskId
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "status": "in_progress",
  "actualStartDate": "2025-01-20",
  "actualHours": 8
}
```

**Status Values:** `not_started`, `in_progress`, `completed`, `blocked`
**Priority Values:** `low`, `medium`, `high`, `urgent`

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "status": "in_progress",
    ...
  }
}
```

---

### Delete Task

```
DELETE /api/projects/:id/tasks/:taskId
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Task deleted"
  }
}
```

---

### Add Task Dependency

```
POST /api/projects/:id/tasks/:taskId/dependencies
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "dependsOnTaskId": "uuid"
}
```

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "taskId": "uuid",
    "dependsOnTaskId": "uuid"
  }
}
```

**Errors:**
| Code | Status | Message |
|------|--------|---------|
| RES_4001 | 404 | Task not found |
| RES_4002 | 409 | Dependency already exists |
| BIZ_5005 | 400 | Cannot create circular dependency |

---

### Remove Task Dependency

```
DELETE /api/projects/:id/tasks/:taskId/dependencies/:depId
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Dependency removed"
  }
}
```

---

## Schedule Milestone Endpoints

Schedule milestones are nested under projects: `/api/projects/:id/milestones`.

### List Milestones

```
GET /api/projects/:id/milestones
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "uuid",
      "projectId": "uuid",
      "phaseId": "uuid",
      "name": "Foundation Complete",
      "description": "All foundation work completed",
      "targetDate": "2025-02-15",
      "actualDate": null,
      "status": "upcoming",
      "createdAt": "2025-12-23T21:00:00.000Z",
      "updatedAt": "2025-12-23T21:00:00.000Z"
    }
  ]
}
```

---

### Create Milestone

```
POST /api/projects/:id/milestones
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "Foundation Complete",
  "phaseId": "uuid",
  "targetDate": "2025-02-15",
  "description": "All foundation work completed"
}
```

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "projectId": "uuid",
    "name": "Foundation Complete",
    ...
  }
}
```

---

### Update Milestone

```
PUT /api/projects/:id/milestones/:milestoneId
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "status": "achieved",
  "actualDate": "2025-02-14"
}
```

**Status Values:** `upcoming`, `achieved`, `missed`

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "status": "achieved",
    "actualDate": "2025-02-14",
    ...
  }
}
```

---

### Delete Milestone

```
DELETE /api/projects/:id/milestones/:milestoneId
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Milestone deleted"
  }
}
```

---

## Budget Item Endpoints

Budget items are nested under projects: `/api/projects/:id/budget`.

### List Budget Items

```
GET /api/projects/:id/budget
Authorization: Bearer <token>
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| categoryId | uuid | Filter by budget category |
| phaseId | uuid | Filter by phase |
| limit | number | Results per page (default: 20, max: 100) |
| offset | number | Pagination offset |

**Response (200 OK):**
```json
{
  "data": {
    "items": [
      {
        "id": "uuid",
        "projectId": "uuid",
        "categoryId": "uuid",
        "categoryName": "Foundation",
        "phaseId": "uuid",
        "phaseName": "Foundation Phase",
        "name": "Concrete",
        "description": "Concrete for footings",
        "estimatedAmount": 15000,
        "actualAmount": 14500,
        "createdAt": "2025-12-23T21:00:00.000Z",
        "updatedAt": "2025-12-23T21:00:00.000Z"
      }
    ],
    "total": 1
  }
}
```

---

### Get Budget Summary

```
GET /api/projects/:id/budget/summary
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "totalEstimated": 500000,
    "totalActual": 125000,
    "remainingBudget": 375000,
    "percentUsed": 25,
    "byCategory": [
      {
        "categoryId": "uuid",
        "categoryName": "Foundation",
        "estimated": 50000,
        "actual": 45000,
        "variance": -5000
      }
    ],
    "byPhase": [
      {
        "phaseId": "uuid",
        "phaseName": "Foundation Phase",
        "estimated": 75000,
        "actual": 70000
      }
    ]
  }
}
```

---

### Create Budget Item

```
POST /api/projects/:id/budget
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "categoryId": "uuid",
  "phaseId": "uuid",
  "name": "Concrete",
  "description": "Concrete for footings",
  "estimatedAmount": 15000,
  "actualAmount": 0
}
```

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "projectId": "uuid",
    "categoryId": "uuid",
    "name": "Concrete",
    ...
  }
}
```

---

### Update Budget Item

```
PUT /api/projects/:id/budget/:itemId
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "actualAmount": 14500,
  "description": "Concrete for footings - final cost"
}
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "actualAmount": 14500,
    ...
  }
}
```

---

### Delete Budget Item

```
DELETE /api/projects/:id/budget/:itemId
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Budget item deleted"
  }
}
```

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
        "purchasePrice": 150000,
        "appraisedValue": 155000,
        "targetArv": 200000,
        "valuationDate": "2025-01-10",
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

**Note:** Valuation fields (`purchasePrice`, `appraisedValue`, `targetArv`) are per-unit values. Multiply by `units` to get project totals.

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
  "description": "24-unit apartment complex",
  "purchasePrice": 150000,
  "appraisedValue": 155000,
  "targetArv": 200000,
  "valuationDate": "2025-01-10"
}
```

**Note:** Valuation fields are per-unit values. For a 24-unit project with $150K purchase price per unit, the total purchase price would be $3.6M.

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

**Side Effects:**
- Automatically populates default schedule template with phases, tasks, and milestones
- Automatically populates default budget template with categories and line items
- Schedule dates are calculated from project start date with realistic durations
- Budget amounts scale based on unit count and unit type

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

### Apply Templates

Re-apply default schedule and/or budget templates to an existing project. Useful if templates were not applied during creation or to reset to defaults.

```
POST /api/projects/:id/templates
Authorization: Bearer <token>
Content-Type: application/json
```

**Requires:** Admin or Manager role in the organization

**Request Body:**
```json
{
  "schedule": true,
  "budget": true
}
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Templates applied: schedule, budget",
    "project": {
      "id": "uuid",
      "name": "Riverside Apartments",
      ...
    }
  }
}
```

**Notes:**
- Schedule template uses the project's start date (or today if not set) to calculate task dates
- Budget template scales costs based on unit type and unit count
- Existing schedule phases/tasks and budget items are NOT deleted; templates add new items

---

## Document Endpoints

Document management endpoints are nested under projects: `/api/projects/:id/documents`.

### Get Upload URL

Get a presigned URL to upload a file directly to R2 storage.

```
POST /api/projects/:id/documents/upload-url
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "filename": "invoice.pdf",
  "mimeType": "application/pdf",
  "sizeBytes": 1048576
}
```

**Response (200 OK):**
```json
{
  "data": {
    "uploadUrl": "https://r2.cloudflarestorage.com/...",
    "storageKey": "documents/uuid/invoice.pdf"
  }
}
```

---

### Analyze Document

Use AI to analyze an uploaded document and suggest metadata.

```
POST /api/projects/:id/documents/analyze
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "storageKey": "documents/uuid/invoice.pdf",
  "mimeType": "application/pdf",
  "originalFilename": "invoice.pdf"
}
```

**Response (200 OK):**
```json
{
  "data": {
    "suggestedName": "Plumbing Invoice - March 2025",
    "suggestedType": "invoice",
    "suggestedDescription": "Invoice from ABC Plumbing for bathroom fixtures",
    "confidence": 0.92,
    "extractedAmount": 15250.00,
    "extractedVendor": "ABC Plumbing",
    "extractedInvoiceNumber": "INV-2025-0342"
  }
}
```

For appraisal documents, additional fields are extracted:
```json
{
  "data": {
    "suggestedName": "Property Appraisal - 123 Main St",
    "suggestedType": "appraisal",
    "suggestedDescription": "Appraisal report for multi-family property",
    "confidence": 0.95,
    "extractedPurchasePrice": 150000,
    "extractedAppraisedValue": 155000,
    "extractedTargetArv": 200000,
    "extractedValuationDate": "2025-01-10"
  }
}
```

**Note:** Extracted valuation fields are per-unit values.

**Document Types:** `estimate`, `contract`, `permit`, `inspection`, `plan`, `invoice`, `receipt`, `photo`, `report`, `insurance`, `warranty`, `appraisal`, `other`

---

### List Documents

```
GET /api/projects/:id/documents
Authorization: Bearer <token>
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| type | string | Filter by document type |
| limit | number | Results per page (default: 20, max: 100) |
| offset | number | Pagination offset |

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "uuid",
      "projectId": "uuid",
      "uploadedBy": "uuid",
      "name": "Plumbing Invoice - March 2025",
      "originalFilename": "invoice.pdf",
      "storageKey": "documents/uuid/invoice.pdf",
      "mimeType": "application/pdf",
      "sizeBytes": 1048576,
      "type": "invoice",
      "description": "Invoice from ABC Plumbing",
      "tags": ["plumbing", "march"],
      "createdAt": "2025-12-25T10:00:00Z",
      "updatedAt": "2025-12-25T10:00:00Z"
    }
  ],
  "total": 1
}
```

---

### Create Document

Create a document record after uploading to storage.

```
POST /api/projects/:id/documents
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "Plumbing Invoice - March 2025",
  "originalFilename": "invoice.pdf",
  "storageKey": "documents/uuid/invoice.pdf",
  "mimeType": "application/pdf",
  "sizeBytes": 1048576,
  "type": "invoice",
  "description": "Invoice from ABC Plumbing",
  "tags": ["plumbing"]
}
```

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "projectId": "uuid",
    "name": "Plumbing Invoice - March 2025",
    ...
  }
}
```

---

### Get Document

```
GET /api/projects/:id/documents/:docId
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "projectId": "uuid",
    "name": "Plumbing Invoice - March 2025",
    ...
  }
}
```

---

### Get Download URL

Get a presigned URL to download/view a document.

```
GET /api/projects/:id/documents/:docId/download
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "downloadUrl": "https://r2.cloudflarestorage.com/...",
    "filename": "invoice.pdf",
    "mimeType": "application/pdf"
  }
}
```

---

### Update Document

```
PATCH /api/projects/:id/documents/:docId
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "Updated Name",
  "type": "receipt",
  "description": "Updated description",
  "tags": ["updated", "tags"]
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

### Delete Document

```
DELETE /api/projects/:id/documents/:docId
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Document deleted"
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
        "assignedTo": "uuid",
        "assigneeName": "John Smith",
        "plannedStartDate": "2025-01-20",
        "plannedEndDate": "2025-01-22",
        "actualStartDate": null,
        "actualEndDate": null,
        "actualHours": null,
        "predecessorTaskId": "uuid",
        "predecessorTaskName": "Excavate foundation",
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
  "plannedStartDate": "2025-01-20",
  "plannedEndDate": "2025-01-22",
  "predecessorTaskId": "uuid"
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
    "predecessorTaskId": "uuid",
    "predecessorTaskName": "Excavate foundation",
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
      "taskId": "uuid",
      "name": "Foundation Complete",
      "description": "All foundation work completed",
      "targetDate": "2025-02-15",
      "actualDate": null,
      "status": "upcoming",
      "isAutoStatus": true,
      "linkedTask": {
        "id": "uuid",
        "name": "Pour foundation",
        "status": "in_progress"
      },
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
  "taskId": "uuid",
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
  "taskId": "uuid",
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

---

## AI Integration

All AI endpoints are prefixed with `/api/ai` and require authentication.

### List Conversations

```
GET /api/ai/conversations
Authorization: Bearer <token>
```

**Query Parameters:**
- `projectId` (optional): Filter by project
- `limit` (optional): Number of results (default: 20, max: 50)
- `offset` (optional): Pagination offset

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "uuid",
      "projectId": "uuid",
      "projectName": "Smith Residence",
      "title": "Budget question",
      "createdAt": "2025-01-15T10:00:00Z",
      "updatedAt": "2025-01-15T10:05:00Z"
    }
  ],
  "total": 10
}
```

### Create Conversation

```
POST /api/ai/conversations
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "projectId": "uuid",
  "title": "Budget Analysis"
}
```

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "projectId": "uuid",
    "title": "Budget Analysis",
    "createdAt": "2025-01-15T10:00:00Z",
    "updatedAt": "2025-01-15T10:00:00Z"
  }
}
```

### Get Conversation

```
GET /api/ai/conversations/:id
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "projectId": "uuid",
    "projectName": "Smith Residence",
    "title": "Budget question",
    "createdAt": "2025-01-15T10:00:00Z",
    "updatedAt": "2025-01-15T10:05:00Z",
    "messages": [
      {
        "id": "uuid",
        "role": "user",
        "content": "How much did we spend on plumbing?",
        "queryType": "spending",
        "sources": null,
        "tokensUsed": null,
        "createdAt": "2025-01-15T10:00:00Z"
      },
      {
        "id": "uuid",
        "role": "assistant",
        "content": "Based on your records, you've spent $15,250 on plumbing...",
        "queryType": "spending",
        "sources": [{"type": "database", "name": "payment_summary"}],
        "tokensUsed": 450,
        "createdAt": "2025-01-15T10:00:05Z"
      }
    ]
  }
}
```

### Delete Conversation

```
DELETE /api/ai/conversations/:id
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Conversation deleted"
  }
}
```

### Send Message

```
POST /api/ai/conversations/:id/messages
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "content": "How much did we spend on plumbing last month?"
}
```

**Response (200 OK):**
```json
{
  "data": {
    "message": {
      "id": "uuid",
      "role": "assistant",
      "content": "Based on your payment records...",
      "queryType": "spending",
      "sources": [
        {"type": "database", "name": "payment_summary"}
      ],
      "tokensUsed": 450,
      "createdAt": "2025-01-15T10:05:00Z"
    },
    "metadata": {
      "intent": "spending",
      "confidence": 0.95,
      "tokensUsed": 800,
      "executionTimeMs": 2500,
      "queryCount": 2
    }
  }
}
```

**Error (429 Rate Limited):**
```json
{
  "error": {
    "code": "SYS_9003",
    "message": "Monthly AI usage limit reached"
  }
}
```

### Submit Feedback

```
POST /api/ai/feedback
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "messageId": "uuid",
  "rating": 5,
  "feedbackText": "Very helpful response"
}
```

**Notes:**
- `rating`: 1 (thumbs down) or 5 (thumbs up)
- `feedbackText`: Optional text feedback

**Response (201 Created):**
```json
{
  "data": {
    "message": "Feedback submitted"
  }
}
```

### List Insights

```
GET /api/ai/insights
Authorization: Bearer <token>
```

**Query Parameters:**
- `projectId` (optional): Filter by project
- `type` (optional): cost_pattern, schedule_risk, efficiency, anomaly, recommendation
- `includeDismissed` (optional): Include dismissed insights (default: false)
- `limit` (optional): Number of results (default: 20, max: 50)
- `offset` (optional): Pagination offset

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "uuid",
      "projectId": "uuid",
      "projectName": "Smith Residence",
      "type": "cost_pattern",
      "title": "Project is 15% over budget",
      "description": "Spending has exceeded the budget by 15%...",
      "data": {"variance": 15, "budget": 100000, "spent": 115000},
      "severity": "warning",
      "isDismissed": false,
      "createdAt": "2025-01-15T06:00:00Z"
    }
  ],
  "total": 5
}
```

### Dismiss Insight

```
POST /api/ai/insights/:id/dismiss
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Insight dismissed"
  }
}
```

### Get Suggestions

```
GET /api/ai/suggestions
Authorization: Bearer <token>
```

**Query Parameters:**
- `projectId` (optional): Get context-aware suggestions for a project

**Response (200 OK):**
```json
{
  "data": [
    "What tasks are overdue?",
    "What's our budget variance?",
    "Show me recent payments",
    "Compare spending by category"
  ]
}
```

### Get Usage

```
GET /api/ai/usage
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "tokensUsed": 45000,
    "tokensLimit": 100000,
    "tokensRemaining": 55000,
    "usagePercentage": 45,
    "resetAt": "2025-02-01T00:00:00Z"
  }
}
```

---

## Capital Risk Endpoints

All risk endpoints require authentication and are prefixed with `/api/risk`. The Capital Risk system analyzes project data to surface where time and money are at risk and recommend interventions.

### Get Risk Dashboard

Get portfolio-wide risk summary with all project risks and top interventions.

```
GET /api/risk/dashboard
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "portfolio": {
      "totalCapitalAtRisk": 125000,
      "projectCount": 5,
      "highRiskCount": 2,
      "criticalRiskCount": 0,
      "riskDistribution": {
        "low": 2,
        "medium": 1,
        "high": 2,
        "critical": 0
      },
      "totalPurchasePrice": 3600000,
      "totalAppraisedValue": 3720000,
      "totalTargetArv": 4800000
    },
    "projectRisks": [
      {
        "project": {
          "id": "uuid",
          "name": "Riverside Apartments",
          "units": 24,
          "unitType": "apartments",
          "purchasePricePerUnit": 150000,
          "appraisedValuePerUnit": 155000,
          "targetArvPerUnit": 200000,
          "purchasePrice": 3600000,
          "appraisedValue": 3720000,
          "targetArv": 4800000
        },
        "metrics": {
          "compositeScore": 62,
          "riskLevel": "high",
          "capitalAtRisk": 75000,
          "trend": "worsening",
          "overdueTasksCount": 5,
          "budgetVariancePercent": 12.5
        }
      }
    ],
    "topInterventions": [
      {
        "id": "uuid",
        "projectId": "uuid",
        "projectName": "Riverside Apartments",
        "type": "overdue_task",
        "severity": "high",
        "priorityRank": 1,
        "title": "5 overdue tasks need attention",
        "description": "Foundation phase has 5 overdue tasks...",
        "recommendedAction": "Review and update task deadlines",
        "capitalImpact": 25000,
        "daysImpact": 14
      }
    ]
  }
}
```

---

### Get Portfolio Metrics

Get aggregated portfolio risk metrics.

```
GET /api/risk/portfolio
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "totalCapitalAtRisk": 125000,
    "projectCount": 5,
    "highRiskCount": 2,
    "criticalRiskCount": 0,
    "riskDistribution": {
      "low": 2,
      "medium": 1,
      "high": 2,
      "critical": 0
    }
  }
}
```

---

### Get Project Risk Detail

Get detailed risk breakdown for a specific project.

```
GET /api/risk/projects/:id
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "project": {
      "id": "uuid",
      "name": "Riverside Apartments",
      "unitType": "apartments",
      "units": 24,
      "status": "active",
      "startDate": "2025-01-15",
      "endDate": "2025-12-31"
    },
    "metrics": {
      "compositeScore": 62,
      "riskLevel": "high",
      "capitalAtRisk": 75000,
      "scheduleRiskScore": 58,
      "budgetRiskScore": 65,
      "trend": "worsening",
      "trendVelocity": 0.15,
      "overdueTasksCount": 5,
      "blockedTasksCount": 2,
      "phaseDelayDays": 7,
      "milestoneSlippageDays": 10,
      "budgetVariancePercent": 12.5,
      "overBudgetCategoriesCount": 3,
      "burnRateDeviation": 18.5,
      "costPerUnitVariance": 8.2,
      "calculatedAt": "2025-12-27T10:00:00Z"
    },
    "interventions": [
      {
        "id": "uuid",
        "type": "overdue_task",
        "severity": "high",
        "priorityRank": 1,
        "title": "5 overdue tasks need attention",
        "description": "Foundation phase has 5 overdue tasks...",
        "recommendedAction": "Review and update task deadlines",
        "entityType": "task",
        "entityId": "uuid",
        "entityName": "Pour footings",
        "capitalImpact": 25000,
        "daysImpact": 14,
        "status": "active",
        "createdAt": "2025-12-27T08:00:00Z"
      }
    ],
    "benchmarkComparison": {
      "scheduleSummary": {
        "elapsed": 45,
        "remaining": 120,
        "progressPercent": 27.3
      },
      "budgetCategories": [
        {
          "name": "Foundation",
          "actual": 55000,
          "actualPercent": 11.0,
          "benchmarkTypical": 10.0,
          "variance": 1.0
        }
      ]
    }
  }
}
```

---

### List Interventions

Get filterable list of all interventions across projects.

```
GET /api/risk/interventions
Authorization: Bearer <token>
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| status | string | Filter by status: `active`, `acknowledged`, `resolved` |
| severity | string | Filter by severity: `low`, `medium`, `high`, `critical` |
| type | string | Filter by type: `overdue_task`, `blocked_task`, `budget_overrun`, `milestone_slip`, `phase_delay`, `burn_rate` |
| limit | number | Results per page (default: 20, max: 100) |
| offset | number | Pagination offset |

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "uuid",
      "projectId": "uuid",
      "type": "overdue_task",
      "severity": "high",
      "priorityRank": 1,
      "title": "5 overdue tasks need attention",
      "description": "Foundation phase has 5 overdue tasks...",
      "recommendedAction": "Review and update task deadlines",
      "entityType": "task",
      "entityId": "uuid",
      "entityName": "Pour footings",
      "capitalImpact": 25000,
      "daysImpact": 14,
      "status": "active",
      "acknowledgedAt": null,
      "createdAt": "2025-12-27T08:00:00Z"
    }
  ],
  "total": 15
}
```

---

### Acknowledge Intervention

Mark an intervention as acknowledged.

```
POST /api/risk/interventions/:id/acknowledge
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "status": "acknowledged",
    "acknowledgedAt": "2025-12-27T10:00:00Z",
    "message": "Intervention acknowledged"
  }
}
```

---

### Recalculate Risk

Trigger risk recalculation for one or all projects.

```
POST /api/risk/recalculate
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body (optional):**
```json
{
  "projectId": "uuid"
}
```

**Response (200 OK) - Single project:**
```json
{
  "data": {
    "projectId": "uuid",
    "compositeScore": 62,
    "riskLevel": "high",
    "capitalAtRisk": 75000,
    "message": "Risk recalculated for project"
  }
}
```

**Response (200 OK) - All projects:**
```json
{
  "data": {
    "projectsCalculated": 5,
    "message": "Risk recalculated for 5 projects"
  }
}
```

---

### Get Builder Benchmarks

Get learned benchmarks from completed projects.

```
GET /api/risk/benchmarks
Authorization: Bearer <token>
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| unitType | string | Filter by unit type: `single_family`, `townhomes`, `condos`, `apartments` |

**Response (200 OK):**
```json
{
  "data": [
    {
      "unitType": "single_family",
      "sampleCount": 5,
      "confidenceLevel": "medium",
      "blendWeight": 0.5,
      "avgDurationDays": 180,
      "avgCostPerUnit": 250000,
      "lastCalculatedAt": "2025-12-27T06:00:00Z"
    }
  ]
}
```

---

### Recalculate Benchmarks

Recalculate builder benchmarks from completed project outcomes.

```
POST /api/risk/benchmarks/recalculate
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "benchmarks": [
      {
        "unitType": "single_family",
        "sampleCount": 5,
        "confidenceLevel": "medium",
        "blendWeight": 0.5,
        "avgDurationDays": 180,
        "avgCostPerUnit": 250000,
        "lastCalculatedAt": "2025-12-27T10:00:00Z"
      }
    ],
    "message": "Builder benchmarks recalculated from project outcomes"
  }
}
```

---

## Reminder Endpoints

All reminder endpoints require authentication and are prefixed with `/api/reminders`.

### List Reminders

```
GET /api/reminders
Authorization: Bearer <token>
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| projectId | uuid | Filter by project |
| upcoming | boolean | Only future reminders (remind_at >= NOW()) |
| dismissed | boolean | Filter by dismissed status |
| limit | number | Results per page (default: 20, max: 100) |
| offset | number | Pagination offset |

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "uuid",
      "title": "Pay contractor",
      "description": "Final payment for foundation work",
      "remindAt": "2025-01-20T09:00:00Z",
      "recurrence": "none",
      "isDismissed": false,
      "relatedEntity": {
        "type": "project",
        "id": "uuid",
        "name": "Smith Residence"
      },
      "createdAt": "2025-01-15T10:00:00Z",
      "updatedAt": "2025-01-15T10:00:00Z"
    }
  ],
  "total": 5
}
```

---

### Create Reminder

```
POST /api/reminders
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "title": "Pay contractor",
  "description": "Final payment for foundation work",
  "remindAt": "2025-01-20T09:00:00Z",
  "recurrence": "none",
  "projectId": "uuid"
}
```

**Recurrence Values:** `none`, `daily`, `weekly`, `biweekly`, `monthly`, `quarterly`, `yearly`

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "title": "Pay contractor",
    "description": "Final payment for foundation work",
    "remindAt": "2025-01-20T09:00:00Z",
    "recurrence": "none",
    "isDismissed": false,
    "createdAt": "2025-01-15T10:00:00Z",
    "updatedAt": "2025-01-15T10:00:00Z"
  }
}
```

---

### Get Reminder

```
GET /api/reminders/:id
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "title": "Pay contractor",
    "description": "Final payment for foundation work",
    "remindAt": "2025-01-20T09:00:00Z",
    "recurrence": "none",
    "isDismissed": false
  }
}
```

---

### Update Reminder

```
PUT /api/reminders/:id
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "title": "Updated title",
  "remindAt": "2025-01-25T09:00:00Z",
  "recurrence": "weekly"
}
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "title": "Updated title",
    "remindAt": "2025-01-25T09:00:00Z",
    "recurrence": "weekly"
  }
}
```

---

### Delete Reminder

```
DELETE /api/reminders/:id
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Reminder deleted"
  }
}
```

---

### Dismiss Reminder

Mark a reminder as dismissed (completed/acknowledged).

```
POST /api/reminders/:id/dismiss
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Reminder dismissed"
  }
}
```

**Notes:**
- Dismissed reminders are excluded by default when listing reminders
- Use `dismissed=true` query parameter to include dismissed reminders
- For recurring reminders, dismissing advances to the next occurrence

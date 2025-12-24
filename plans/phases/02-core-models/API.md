# Phase 2: API Endpoints

Organization, project, and activity management endpoints.

---

## Organizations

### GET /organizations

List organizations the current user belongs to.

**Response `200 OK`**

```typescript
{
  data: [{
    id: string;
    name: string;
    role: "admin" | "member";
    memberCount: number;
    projectCount: number;
    isDefault: boolean;
  }]
}
```

---

### GET /organizations/:id

Get organization details.

**Response `200 OK`**

```typescript
{
  data: {
    id: string;
    name: string;
    isDefault: boolean;
    createdAt: string;
    memberCount: number;
    projectCount: number;
  }
}
```

---

### PUT /organizations/:id

Update organization. Requires admin role.

**Request**

```typescript
{
  name?: string;  // min 2 chars
}
```

**Response `200 OK`**

```typescript
{
  data: {
    id: string;
    name: string;
    updatedAt: string;
  }
}
```

---

### GET /organizations/:id/members

List organization members. Requires admin role.

**Query Parameters**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| limit | number | 50 | Max results |
| offset | number | 0 | Pagination offset |

**Response `200 OK`**

```typescript
{
  data: [{
    id: string;           // membership id
    user: {
      id: string;
      name: string;
      email: string;
    };
    role: "admin" | "member";
    joinedAt: string;
    isActive: boolean;
  }],
  total: number
}
```

---

### POST /organizations/:id/members/invite

Invite new member. Requires admin role.

**Request**

```typescript
{
  email: string;       // Required, valid email
  role: "admin" | "member";  // Required
  message?: string;    // Optional personal note
}
```

**Response `201 Created`**

```typescript
{
  data: {
    id: string;
    email: string;
    role: string;
    expiresAt: string;
  }
}
```

**Errors**

| Code | Status | Message |
|------|--------|---------|
| RES_4002 | 409 | User is already a member |

**Side Effects**

- Creates invitation record with token
- Sends invitation email

---

### PUT /organizations/:id/members/:userId/role

Change member role. Requires admin role.

**Request**

```typescript
{
  role: "admin" | "member";
}
```

**Response `200 OK`**

```typescript
{
  data: {
    userId: string;
    role: string;
    updatedAt: string;
  }
}
```

**Errors**

| Code | Status | Message |
|------|--------|---------|
| BIZ_5001 | 400 | Cannot demote the last admin |

---

### DELETE /organizations/:id/members/:userId

Remove member. Requires admin role.

**Response `200 OK`**

```typescript
{
  data: {
    message: "Member removed successfully"
  }
}
```

**Errors**

| Code | Status | Message |
|------|--------|---------|
| BIZ_5001 | 400 | Cannot remove yourself |

---

## Invitations

### GET /invitations/validate

Validate invitation token.

**Query Parameters**

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| token | string | Yes | Invitation token |

**Response `200 OK`**

```typescript
{
  data: {
    valid: boolean;
    email: string;
    organization: {
      id: string;
      name: string;
    };
    invitedBy: {
      name: string;
    };
    expiresAt: string;
  }
}
```

---

### POST /organizations/:id/members/accept

Accept invitation. Requires authenticated user matching invitation email.

**Request**

```typescript
{
  token: string;
}
```

**Response `200 OK`**

```typescript
{
  data: {
    organization: {
      id: string;
      name: string;
    };
    role: string;
  }
}
```

**Errors**

| Code | Status | Message |
|------|--------|---------|
| BIZ_5003 | 400 | Invitation expired or invalid |

---

### POST /organizations/:id/invitations/:invitationId/resend

Resend invitation email. Requires admin role.

**Response `200 OK`**

```typescript
{
  data: {
    message: "Invitation resent",
    expiresAt: string;
  }
}
```

---

### DELETE /organizations/:id/invitations/:invitationId

Cancel invitation. Requires admin role.

**Response `200 OK`**

```typescript
{
  data: {
    message: "Invitation cancelled"
  }
}
```

---

## Projects

### GET /projects

List projects for current builder.

**Query Parameters**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| status | string | - | Filter by status |
| organizationId | string | - | Filter by organization |
| unitType | string | - | Filter by unit type |
| search | string | - | Search name/address |
| sortBy | string | created_at | Sort field |
| sortOrder | string | desc | asc or desc |
| limit | number | 20 | Max results |
| offset | number | 0 | Pagination offset |

**Response `200 OK`**

```typescript
{
  data: [{
    id: string;
    name: string;
    address: string | null;
    unitType: string;
    units: number;
    status: string;
    organization: {
      id: string;
      name: string;
    };
    startDate: string | null;
    endDate: string | null;
    createdAt: string;
  }],
  total: number;
  limit: number;
  offset: number;
}
```

---

### POST /projects

Create new project.

**Request**

```typescript
{
  name: string;           // Required, min 2 chars
  organizationId: string; // Required, valid org UUID
  address?: string;
  unitType: string;       // Required: single_family, townhomes, condos, apartments
  units: number;          // Required, min 1
  lotSizeAcres?: number;
  startDate?: string;     // YYYY-MM-DD
  endDate?: string;       // YYYY-MM-DD
  description?: string;
}
```

**Response `201 Created`**

```typescript
{
  data: {
    id: string;
    name: string;
    address: string | null;
    unitType: string;
    units: number;
    lotSizeAcres: number | null;
    status: "planning";
    organizationId: string;
    startDate: string | null;
    endDate: string | null;
    description: string | null;
    createdAt: string;
  }
}
```

---

### GET /projects/:id

Get project details.

**Response `200 OK`**

```typescript
{
  data: {
    id: string;
    name: string;
    address: string | null;
    unitType: string;
    units: number;
    lotSizeAcres: number | null;
    status: string;
    organization: {
      id: string;
      name: string;
    };
    startDate: string | null;
    endDate: string | null;
    description: string | null;
    createdAt: string;
    updatedAt: string;
  }
}
```

---

### PUT /projects/:id

Update project.

**Request**

```typescript
{
  name?: string;
  address?: string;
  unitType?: string;
  units?: number;
  lotSizeAcres?: number;
  status?: string;
  startDate?: string;
  endDate?: string;
  description?: string;
}
```

**Response `200 OK`**

```typescript
{
  data: {
    id: string;
    // ... all project fields
    updatedAt: string;
  }
}
```

---

### DELETE /projects/:id

Soft delete (archive) project.

**Response `200 OK`**

```typescript
{
  data: {
    message: "Project archived successfully"
  }
}
```

---

### GET /projects/:id/summary

Get calculated project summary.

**Response `200 OK`**

```typescript
{
  data: {
    id: string;
    name: string;
    status: string;
    units: number;
    budget: {
      totalEstimated: number;
      totalActual: number;
      variance: number;
      variancePercent: number;
    };
    schedule: {
      totalTasks: number;
      completedTasks: number;
      overdueTasks: number;
      progressPercent: number;
    };
    milestones: {
      total: number;
      achieved: number;
      upcoming: number;
      nextMilestone: {
        name: string;
        targetDate: string;
      } | null;
    };
    payments: {
      totalPaid: number;
      outstandingInvoices: number;
      outstandingAmount: number;
    };
    costPerUnit: number;
    daysRemaining: number | null;
  }
}
```

---

### GET /projects/compare

Compare multiple projects.

**Query Parameters**

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| ids | string | Yes | Comma-separated project IDs |

**Response `200 OK`**

```typescript
{
  data: [{
    id: string;
    name: string;
    units: number;
    unitType: string;
    status: string;
    totalBudget: number;
    totalSpent: number;
    costPerUnit: number;
    progressPercent: number;
    startDate: string | null;
    endDate: string | null;
  }]
}
```

---

## Activity

### GET /projects/:id/activity

Get activity log for a project.

**Query Parameters**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| limit | number | 50 | Max results |
| offset | number | 0 | Pagination offset |

**Response `200 OK`**

```typescript
{
  data: [{
    id: string;
    user: {
      id: string;
      name: string;
    } | null;
    entityType: string;
    entityId: string;
    action: string;
    changes: Record<string, { from: unknown; to: unknown }> | null;
    createdAt: string;
  }],
  total: number
}
```

---

### GET /activity

Get builder-wide activity log.

**Query Parameters**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| projectId | string | - | Filter by project |
| entityType | string | - | Filter by entity type |
| limit | number | 50 | Max results |
| offset | number | 0 | Pagination offset |

**Response `200 OK`**

```typescript
{
  data: [{
    id: string;
    user: {
      id: string;
      name: string;
    } | null;
    project: {
      id: string;
      name: string;
    } | null;
    entityType: string;
    entityId: string;
    action: string;
    changes: Record<string, { from: unknown; to: unknown }> | null;
    createdAt: string;
  }],
  total: number
}
```

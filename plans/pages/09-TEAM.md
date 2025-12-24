# Team Management

Admin-only page for managing organization members and invitations.

Route: `/team`

Access: Admin role only

---

## Full Page Layout

```
+-----------------------------------------------------------------------------------+
| [Logo] HarvestIQ     [_____Search_____]              [AI] [Bell(3)] [Avatar]      |
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  TEAM MANAGEMENT                                   [+ Invite Member]   |
| Projects |                                                                        |
| Tasks    |  Organization: [Acme Builders - Main               v]                  |
|          |                                                                        |
|          |  [Members]  [Pending Invitations]                                      |
| ──────── |  ============================================================          |
| Team     |                                                                        |
| Settings |  MEMBERS (5)                                                           |
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
|          |  | Lisa Chen       | lisa@acme.com          | Member | [...]     |    |
|          |  |                 | Joined Dec 15, 2024    |        |           |    |
|          |  +---------------------------------------------------------------+    |
|          |  | Bob Martinez    | bob@acme.com           | Member | [...]     |    |
|          |  |                 | Joined Dec 18, 2024    |        |           |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
+-----------------------------------------------------------------------------------+
```

---

## Members Tab

### Table Columns

| Column | Field | Description |
|--------|-------|-------------|
| Name | user.name | User's display name |
| Email | user.email + joined_at | Email and join date |
| Role | organization_members.role | Admin or Member |
| Actions | - | Role change, remove |

### Row Actions Menu

```
+------------------+
| Change Role      |
|   > Admin        |
|   > Member       |
+------------------+
| Remove from Team |
+------------------+
```

- Cannot remove yourself
- Cannot demote the last admin

### API Needed

```
GET /organizations/:id/members?limit=50&offset=0

Response:
{
  data: [{
    id: uuid,  // membership id
    user: { id, name, email },
    role: enum,
    joined_at: timestamp,
    is_active: boolean
  }],
  total: number
}

PUT /organizations/:id/members/:userId/role
Body: { role: "admin" | "member" }

DELETE /organizations/:id/members/:userId
```

---

## Pending Invitations Tab

```
|          |  [Members]  [Pending Invitations]                                      |
|          |  ============================================================          |
|          |                                                                        |
|          |  PENDING INVITATIONS (2)                                               |
|          |  +---------------------------------------------------------------+    |
|          |  | Email                   | Role   | Invited    | Actions       |    |
|          |  +---------------------------------------------------------------+    |
|          |  | newuser@example.com     | Member | Dec 20     | [Resend][Cancel]|   |
|          |  | Invited by: Joey Smith  |        | Expires: Dec 27            |    |
|          |  +---------------------------------------------------------------+    |
|          |  | another@example.com     | Admin  | Dec 22     | [Resend][Cancel]|   |
|          |  | Invited by: Sarah Johnson        | Expires: Dec 29            |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  No expired invitations                                                |
|          |                                                                        |
```

### API Needed

```
GET /organizations/:id/invitations?status=pending

Response:
{
  data: [{
    id: uuid,
    email: string,
    role: enum,
    invited_by: { id, name },
    created_at: timestamp,
    expires_at: timestamp
  }]
}

POST /organizations/:id/invitations/:id/resend

DELETE /organizations/:id/invitations/:id  // Cancel
```

---

## Invite Member Modal

```
+------------------------------------------------------------------+
|  Invite Team Member                                       [X]    |
+------------------------------------------------------------------+
|                                                                  |
|  Email Address *                                                 |
|  [_____________________________________________________]         |
|                                                                  |
|  Role                                                            |
|  ( ) Admin - Full access, can manage team                        |
|  (x) Member - Access to assigned projects only                   |
|                                                                  |
|  Personal Message (optional)                                     |
|  [_____________________________________________________]         |
|  [_____________________________________________________]         |
|                                                                  |
|  Invitation expires in 7 days.                                   |
|                                                                  |
|                                     [Cancel]  [Send Invitation]  |
+------------------------------------------------------------------+
```

### Form Fields

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| email | string | Yes | Valid email, not already a member |
| role | enum | Yes | admin, member |
| message | string | No | Optional personal note in email |

### API Needed

```
POST /organizations/:id/members/invite
Body: {
  email: string,
  role: string,
  message?: string
}

Response:
{
  id: uuid,
  email: string,
  role: string,
  expires_at: timestamp
}
```

---

## Accept Invitation Flow

When invited user clicks email link:

Route: `/invite/accept?token=X`

```
+------------------------------------------------------------------+
|                                                                  |
|                         [HarvestIQ Logo]                         |
|                                                                  |
|                    +-------------------------+                   |
|                    | You've Been Invited     |                   |
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | Joey Smith invited you  |                   |
|                    | to join Acme Builders   |                   |
|                    | on HarvestIQ.           |                   |
|                    |                         |                   |
|                    | [  Accept Invitation  ] |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
|                                                                  |
+------------------------------------------------------------------+
```

### Existing User Flow
1. Click accept → Redirects to login if not logged in
2. After login → Added to organization
3. Redirects to dashboard

### New User Flow
1. Click accept → Redirects to registration with email pre-filled
2. After registration → Added to organization
3. Redirects to dashboard

### API Needed

```
GET /invitations/validate?token=X

Response:
{
  valid: boolean,
  email: string,
  organization: { id, name },
  invited_by: { name },
  expires_at: timestamp
}

POST /organizations/:id/members/accept
Body: { token: string }

Response:
{
  organization: { id, name },
  role: string
}
```

---

## Change Role Confirmation

```
+------------------------------------------------------------------+
|  Change Role                                              [X]    |
+------------------------------------------------------------------+
|                                                                  |
|  Change Mike Wilson's role to Admin?                             |
|                                                                  |
|  Admin users can:                                                |
|  - Access all projects                                           |
|  - Manage team members                                           |
|  - Change organization settings                                  |
|                                                                  |
|                                        [Cancel]  [Change Role]   |
+------------------------------------------------------------------+
```

---

## Remove Member Confirmation

```
+------------------------------------------------------------------+
|  Remove Team Member                                       [X]    |
+------------------------------------------------------------------+
|                                                                  |
|  Remove Mike Wilson from Acme Builders?                          |
|                                                                  |
|  They will lose access to all projects and data in this          |
|  organization. This action cannot be undone.                     |
|                                                                  |
|                                           [Cancel]  [Remove]     |
+------------------------------------------------------------------+
```

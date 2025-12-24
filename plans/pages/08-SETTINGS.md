# Settings

User profile and preferences.

Route: `/settings`

---

## Full Page Layout

```
+-----------------------------------------------------------------------------------+
| [Logo] HarvestIQ     [_____Search_____]              [AI] [Bell(3)] [Avatar]      |
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  SETTINGS                                                              |
| Projects |                                                                        |
| Tasks    |  [Profile]  [Notifications]  [Security]                                |
|          |  ============================================================          |
|          |                                                                        |
| ──────── |                                                                        |
| Team     |                                                                        |
| Settings |                                                                        |
|          |                                                                        |
+-----------------------------------------------------------------------------------+
```

---

## Profile Tab

Route: `/settings` or `/settings/profile`

```
|          |                                                                        |
|          |  PROFILE                                                               |
|          |                                                                        |
|          |  +---------------------------------------------------------------+    |
|          |  |                                                               |    |
|          |  |  [Avatar]     Joey Smith                                      |    |
|          |  |  [Change]     joey@acmebuilders.com                           |    |
|          |  |               Acme Builders Inc.                              |    |
|          |  |                                                               |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  PERSONAL INFORMATION                                                  |
|          |  +---------------------------------------------------------------+    |
|          |  |                                                               |    |
|          |  |  Name                                                         |    |
|          |  |  [Joey Smith_________________________________]                |    |
|          |  |                                                               |    |
|          |  |  Email                                                        |    |
|          |  |  [joey@acmebuilders.com_____________________]                 |    |
|          |  |                                                               |    |
|          |  |  Phone                                                        |    |
|          |  |  [555-123-4567______________________________]                 |    |
|          |  |                                                               |    |
|          |  |                                              [Save Changes]   |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
```

### API Needed

```
GET /users/me

Response:
{
  id, name, email, phone,
  notification_preferences: object,
  email_verified_at: timestamp,
  last_login_at: timestamp,
  builder: { id, name },
  organizations: [{ id, name, role }]
}

PUT /users/me
Body: { name?, phone? }

// Email change requires verification flow
PUT /users/me/email
Body: { email: string, password: string }
// Sends verification to new email
```

---

## Notifications Tab

Route: `/settings/notifications`

```
|          |                                                                        |
|          |  [Profile]  [Notifications]  [Security]                                |
|          |  ============================================================          |
|          |                                                                        |
|          |  NOTIFICATION CHANNELS                                                 |
|          |  +---------------------------------------------------------------+    |
|          |  |                                                               |    |
|          |  |  Email Notifications                           [x] Enabled   |    |
|          |  |  Receive notifications at joey@acmebuilders.com              |    |
|          |  |                                                               |    |
|          |  |  SMS Notifications                             [ ] Disabled  |    |
|          |  |  Receive text messages at 555-123-4567                       |    |
|          |  |                                                               |    |
|          |  |  In-App Notifications                          [x] Enabled   |    |
|          |  |  See notifications in the app                                |    |
|          |  |                                                               |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  NOTIFICATION PREFERENCES                                              |
|          |  +---------------------------------------------------------------+    |
|          |  |                                  | Email | SMS | In-App       |    |
|          |  |----------------------------------+-------+-----+--------------|    |
|          |  | Task assigned to me              |  [ ]  | [ ] |  [x]         |    |
|          |  | Task due soon                    |  [x]  | [ ] |  [x]         |    |
|          |  | Task overdue                     |  [x]  | [x] |  [x]         |    |
|          |  | Milestone approaching            |  [x]  | [ ] |  [x]         |    |
|          |  | Invoice due                      |  [x]  | [ ] |  [ ]         |    |
|          |  | Reminders                        |  [x]  | [x] |  [x]         |    |
|          |  | Budget alerts                    |  [x]  | [ ] |  [x]         |    |
|          |  | System notifications             |  [ ]  | [ ] |  [x]         |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |                                               [Save Preferences]       |
|          |                                                                        |
```

### API Needed

```
GET /users/me/notification-preferences

Response:
{
  channels: {
    email: boolean,
    sms: boolean,
    in_app: boolean
  },
  types: {
    task_assigned: ["in_app"],
    task_due: ["email", "in_app"],
    task_overdue: ["email", "sms", "in_app"],
    milestone_approaching: ["email", "in_app"],
    invoice_due: ["email"],
    reminder: ["email", "sms", "in_app"],
    budget_alert: ["email", "in_app"],
    system: ["in_app"]
  }
}

PUT /users/me/notification-preferences
Body: {
  channels: { email: true, sms: false, in_app: true },
  types: {
    task_assigned: ["in_app"],
    ...
  }
}
```

---

## Security Tab

Route: `/settings/security`

```
|          |                                                                        |
|          |  [Profile]  [Notifications]  [Security]                                |
|          |  ============================================================          |
|          |                                                                        |
|          |  CHANGE PASSWORD                                                       |
|          |  +---------------------------------------------------------------+    |
|          |  |                                                               |    |
|          |  |  Current Password                                             |    |
|          |  |  [________________________________________]                   |    |
|          |  |                                                               |    |
|          |  |  New Password                                                 |    |
|          |  |  [________________________________________]                   |    |
|          |  |  Minimum 8 characters                                         |    |
|          |  |                                                               |    |
|          |  |  Confirm New Password                                         |    |
|          |  |  [________________________________________]                   |    |
|          |  |                                                               |    |
|          |  |                                         [Change Password]     |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
|          |  ACTIVE SESSIONS                                                       |
|          |  +---------------------------------------------------------------+    |
|          |  |                                                               |    |
|          |  |  Current Session                                              |    |
|          |  |  Chrome on macOS - San Francisco, CA                          |    |
|          |  |  Last active: Now                                             |    |
|          |  |                                                               |    |
|          |  |  Other Session                                   [Revoke]     |    |
|          |  |  Safari on iPhone - San Francisco, CA                         |    |
|          |  |  Last active: 2 hours ago                                     |    |
|          |  |                                                               |    |
|          |  |                                      [Revoke All Other]       |    |
|          |  +---------------------------------------------------------------+    |
|          |                                                                        |
```

### API Needed

```
PUT /users/me/password
Body: {
  current_password: string,
  new_password: string
}

Response: { message: "Password changed successfully" }
// Also invalidates all other sessions

GET /users/me/sessions

Response:
{
  data: [{
    id: uuid,
    device_info: string,
    created_at: timestamp,
    last_used_at: timestamp,
    is_current: boolean
  }]
}

DELETE /users/me/sessions/:id  // Revoke specific session
DELETE /users/me/sessions      // Revoke all except current
```

---

## Form Validation

| Field | Validation |
|-------|------------|
| name | Required, max 255 chars |
| phone | Optional, valid phone format |
| current_password | Required for password change |
| new_password | Min 8 chars |
| confirm_password | Must match new_password |

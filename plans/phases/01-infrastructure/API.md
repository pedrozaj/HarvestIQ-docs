# Phase 1: API Endpoints

Authentication, user management, and builder endpoints.

---

## Base URL

```
https://harvestiq-backend-production.up.railway.app/api
```

---

## Authentication Headers

After login, include JWT in requests:

```
Authorization: Bearer <jwt_token>
```

Or use httpOnly cookie (preferred for web):

```
Cookie: access_token=<jwt_token>
```

---

## POST /auth/register

Create a new builder account with initial user and organization. Sends verification email.

### Request

```typescript
{
  company_name: string;  // Required, min 2 chars (becomes Builder name)
  name: string;          // Required, min 2 chars (user's name)
  email: string;         // Required, valid email, unique
  phone?: string;        // Optional
  password: string;      // Required, min 8 chars, 1 upper, 1 lower, 1 number
}
```

### Response `201 Created`

```typescript
{
  data: {
    message: "Verification email sent"
  }
}
```

### Side Effects

1. Creates builder record with `subscription_status: 'trial'`
2. Creates user record with `email_verified_at: null`
3. Creates default organization
4. Creates organization membership with `role: 'admin'`
5. Sends verification email with token

### Errors

| Code | Status | Message |
|------|--------|---------|
| AUTH_1005 | 409 | Email already registered |
| AUTH_1006 | 400 | Password does not meet requirements |
| VAL_3001 | 400 | Validation failed |

### Example

```bash
curl -X POST https://api.example.com/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "company_name": "Acme Builders",
    "name": "Joey Smith",
    "email": "joey@acmebuilders.com",
    "password": "SecurePass123"
  }'
```

---

## POST /auth/verify-email

Verify email address with token from verification email.

### Request

```typescript
{
  token: string;  // Required, from email link
}
```

### Response `200 OK`

```typescript
{
  data: {
    message: "Email verified successfully"
  }
}
```

### Side Effects

- Sets `email_verified_at` on user record

### Errors

| Code | Status | Message |
|------|--------|---------|
| AUTH_1003 | 400 | Invalid or expired verification token |

---

## POST /auth/resend-verification

Resend verification email.

### Request

```typescript
{
  email: string;  // Required
}
```

### Response `200 OK`

Always returns success (prevents email enumeration):

```typescript
{
  data: {
    message: "If account exists and is unverified, verification email sent"
  }
}
```

---

## POST /auth/login

Authenticate user and receive tokens. Requires verified email.

### Request

```typescript
{
  email: string;     // Required
  password: string;  // Required
}
```

### Response `200 OK`

```typescript
{
  data: {
    user: {
      id: string;
      email: string;
      name: string;
      builderId: string;
    };
    accessToken: string;
  }
}
```

### Cookie Response

Sets httpOnly cookies:

```
Set-Cookie: access_token=<jwt>; HttpOnly; Secure; SameSite=Strict; Max-Age=900
Set-Cookie: refresh_token=<jwt>; HttpOnly; Secure; SameSite=Strict; Path=/api/auth/refresh; Max-Age=604800
```

### Errors

| Code | Status | Message |
|------|--------|---------|
| AUTH_1001 | 401 | Invalid email or password |
| AUTH_1007 | 403 | Email not verified |
| AUTH_1008 | 423 | Account locked. Try again in X minutes |

### Account Lockout

- After 5 failed login attempts, account is locked for 15 minutes
- `failed_login_attempts` counter is reset on successful login
- `locked_until` timestamp is checked before allowing login

---

## POST /auth/logout

Invalidate current session and revoke refresh token.

### Request

No body required. Uses cookie or Authorization header.

### Response `200 OK`

```typescript
{
  data: {
    message: "Logged out successfully"
  }
}
```

### Cookie Response

Clears auth cookies:

```
Set-Cookie: access_token=; HttpOnly; Secure; SameSite=Strict; Max-Age=0
Set-Cookie: refresh_token=; HttpOnly; Secure; SameSite=Strict; Max-Age=0
```

### Side Effects

- Sets `revoked_at` on refresh token record

---

## POST /auth/refresh

Get new access token using refresh token. Implements token rotation.

### Request

Refresh token from cookie or body:

```typescript
{
  refreshToken?: string;  // Optional if using cookies
}
```

### Response `200 OK`

```typescript
{
  data: {
    accessToken: string;
  }
}
```

### Side Effects

- Revokes old refresh token
- Creates new refresh token (rotation)
- Updates cookies with new tokens

### Errors

| Code | Status | Message |
|------|--------|---------|
| AUTH_1004 | 401 | Refresh token expired or invalid |
| AUTH_1009 | 403 | Subscription inactive |

### Subscription Check

Token refresh validates builder subscription status:
- `cancelled` or `past_due` → Returns 403

---

## POST /auth/forgot-password

Initiate password reset flow.

### Request

```typescript
{
  email: string;  // Required
}
```

### Response `200 OK`

Always returns success (prevents email enumeration):

```typescript
{
  data: {
    message: "If an account exists, a reset email has been sent"
  }
}
```

### Side Effects

- Generates password reset token (SHA-256 hashed before storage)
- Sets `password_reset_expires_at` to 1 hour from now
- Sends email with reset link

---

## POST /auth/reset-password

Complete password reset with token.

### Request

```typescript
{
  token: string;      // Required, from email link
  password: string;   // Required, must meet requirements
}
```

### Response `200 OK`

```typescript
{
  data: {
    message: "Password reset successfully"
  }
}
```

### Side Effects

- Updates password hash
- Clears `password_reset_token` and `password_reset_expires_at`
- Revokes all refresh tokens for user (invalidates all sessions)

### Errors

| Code | Status | Message |
|------|--------|---------|
| AUTH_1003 | 400 | Invalid or expired reset token |
| AUTH_1006 | 400 | Password does not meet requirements |

---

## GET /users/me

Get current authenticated user.

### Headers

```
Authorization: Bearer <jwt_token>
```

### Response `200 OK`

```typescript
{
  data: {
    id: string;
    email: string;
    name: string;
    phone: string | null;
    timezone: string;
    emailVerifiedAt: string | null;
    lastLoginAt: string | null;
    builder: {
      id: string;
      name: string;
      subscriptionPlan: string;
      subscriptionStatus: string;
    };
    organizations: [{
      id: string;
      name: string;
      role: "admin" | "member";
    }];
    createdAt: string;
  }
}
```

### Errors

| Code | Status | Message |
|------|--------|---------|
| AUTH_1002 | 401 | Token expired |
| AUTH_1003 | 401 | Invalid token |

---

## PUT /users/me

Update current user profile.

### Request

```typescript
{
  name?: string;      // min 2 chars
  phone?: string;     // optional
  timezone?: string;  // valid timezone
}
```

### Response `200 OK`

```typescript
{
  data: {
    id: string;
    email: string;
    name: string;
    phone: string | null;
    timezone: string;
    updatedAt: string;
  }
}
```

---

## PUT /users/me/password

Change password for authenticated user.

### Request

```typescript
{
  currentPassword: string;  // Required
  newPassword: string;      // Required, must meet requirements
}
```

### Response `200 OK`

```typescript
{
  data: {
    message: "Password changed successfully"
  }
}
```

### Side Effects

- Revokes all other refresh tokens (invalidates other sessions)
- Current session remains active

### Errors

| Code | Status | Message |
|------|--------|---------|
| AUTH_1001 | 400 | Current password is incorrect |
| AUTH_1006 | 400 | New password does not meet requirements |

---

## PUT /users/me/notification-preferences

Update notification preferences.

### Request

```typescript
{
  channels: {
    email: boolean;
    sms: boolean;
    in_app: boolean;
  };
  types: {
    reminder: string[];      // e.g., ["email", "in_app"]
    task_assigned: string[];
    task_due: string[];
    milestone_approaching: string[];
    invoice_due: string[];
    system: string[];
  };
}
```

### Response `200 OK`

Returns updated preferences object.

---

## GET /users/me/sessions

List active sessions for current user.

### Response `200 OK`

```typescript
{
  data: [{
    id: string;
    deviceInfo: string | null;
    createdAt: string;
    isCurrent: boolean;
  }]
}
```

---

## DELETE /users/me/sessions/:id

Revoke a specific session.

### Response `200 OK`

```typescript
{
  data: {
    message: "Session revoked"
  }
}
```

### Errors

| Code | Status | Message |
|------|--------|---------|
| RES_4001 | 404 | Session not found |
| AUTHZ_2002 | 403 | Cannot revoke current session |

---

## DELETE /users/me/sessions

Revoke all sessions except current.

### Response `200 OK`

```typescript
{
  data: {
    revokedCount: number;
  }
}
```

---

## GET /builder

Get current builder details.

### Response `200 OK`

```typescript
{
  data: {
    id: string;
    name: string;
    subscriptionStatus: "active" | "trial" | "cancelled" | "past_due";
    subscriptionPlan: "free" | "pro" | "enterprise";
    trialEndsAt: string | null;
    storageUsedBytes: number;
    storageLimitBytes: number;
    aiTokensUsedMonth: number;
    aiTokensLimitMonth: number;
    createdAt: string;
  }
}
```

---

## PUT /builder

Update builder settings. (Admin only)

### Request

```typescript
{
  name?: string;
  billingEmail?: string;
}
```

### Response `200 OK`

Returns updated builder object.

### Errors

| Code | Status | Message |
|------|--------|---------|
| AUTHZ_2003 | 403 | Admin role required |

---

## JWT Token Structure

### Access Token Payload

```typescript
{
  sub: string;        // User ID
  email: string;
  builderId: string;
  iat: number;        // Issued at
  exp: number;        // Expires (15 minutes)
}
```

### Refresh Token Payload

```typescript
{
  sub: string;        // User ID
  type: "refresh";
  jti: string;        // Token ID (for revocation lookup)
  iat: number;
  exp: number;        // Expires (7 days)
}
```

---

## Rate Limiting

| Endpoint | Limit | Key |
|----------|-------|-----|
| POST /auth/login | 10/minute | IP |
| POST /auth/register | 5/minute | IP |
| POST /auth/forgot-password | 3/minute | email |
| POST /auth/refresh | 30/minute | user |
| All other endpoints | 100/minute | user |

Rate limit headers included in responses:
- `X-RateLimit-Limit`
- `X-RateLimit-Remaining`
- `X-RateLimit-Reset`

---

## Security Considerations

1. **Password hashing**: Argon2id
2. **Token storage**: httpOnly cookies for web, secure storage for mobile
3. **Refresh token storage**: SHA-256 hashed before database storage
4. **HTTPS only**: All endpoints require HTTPS
5. **CORS**: Restrict to allowed origins
6. **Rate limiting**: Prevent brute force attacks
7. **Token rotation**: New refresh token on each refresh
8. **Account lockout**: 5 failed attempts → 15 minute lockout
9. **Session invalidation**: Password change revokes all other sessions

---

## Error Response Format

All errors return:

```typescript
{
  error: {
    code: string;     // Machine-readable error code
    message: string;  // Human-readable message
    details?: object; // Additional context (validation errors, etc.)
  }
}
```

Example validation error:

```json
{
  "error": {
    "code": "VAL_3001",
    "message": "Validation failed",
    "details": {
      "fields": {
        "email": "Must be a valid email address",
        "password": "Must be at least 8 characters"
      }
    }
  }
}
```

# Login & Register Pages

Authentication entry points.

---

## Login Page

Route: `/` or `/login`

```
+------------------------------------------------------------------+
|                                                                  |
|                         [HarvestIQ Logo]                         |
|                                                                  |
|                    +-------------------------+                   |
|                    |        Sign In          |                   |
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | Email                   |                   |
|                    | [___________________]   |                   |
|                    |                         |                   |
|                    | Password                |                   |
|                    | [___________________]   |                   |
|                    |                         |                   |
|                    | [Forgot Password?]      |                   |
|                    |                         |                   |
|                    | [      Sign In      ]   |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
|                                                                  |
|                    Don't have an account?                        |
|                    [Create one]                                  |
|                                                                  |
+------------------------------------------------------------------+
```

### Form Fields

| Field | Type | Validation |
|-------|------|------------|
| email | string | Required, valid email format |
| password | string | Required, min 8 chars |

### API Needed

```
POST /auth/login
  body: { email: string, password: string }
  response: { accessToken: string, user: { id, name, email, role } }
  cookies: refresh_token (HTTP-only)
```

### Error States

- Invalid credentials → "Invalid email or password"
- Account locked → "Account locked. Try again in 15 minutes"
- Email not verified → "Please verify your email first" + resend link

---

## Register Page

Route: `/register`

```
+------------------------------------------------------------------+
|                                                                  |
|                         [HarvestIQ Logo]                         |
|                                                                  |
|                    +-------------------------+                   |
|                    |    Create Account       |                   |
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | Company Name            |                   |
|                    | [___________________]   |                   |
|                    |                         |                   |
|                    | Your Name               |                   |
|                    | [___________________]   |                   |
|                    |                         |                   |
|                    | Email                   |                   |
|                    | [___________________]   |                   |
|                    |                         |                   |
|                    | Phone (optional)        |                   |
|                    | [___________________]   |                   |
|                    |                         |                   |
|                    | Password                |                   |
|                    | [___________________]   |                   |
|                    |                         |                   |
|                    | Confirm Password        |                   |
|                    | [___________________]   |                   |
|                    |                         |                   |
|                    | [    Create Account   ] |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
|                                                                  |
|                    Already have an account?                      |
|                    [Sign in]                                     |
|                                                                  |
+------------------------------------------------------------------+
```

### Form Fields

| Field | Type | Validation |
|-------|------|------------|
| company_name | string | Required (becomes Builder name) |
| name | string | Required (user's name) |
| email | string | Required, valid email, unique |
| phone | string | Optional |
| password | string | Required, min 8 chars |
| confirm_password | string | Must match password |

### API Needed

```
POST /auth/register
  body: {
    company_name: string,
    name: string,
    email: string,
    phone?: string,
    password: string
  }
  response: { message: "Verification email sent" }
```

### Flow

1. Submit form → Creates Builder + Organization + Admin User
2. Sends verification email
3. Shows "Check your email" message
4. User clicks link → `/auth/verify-email?token=X`
5. Redirects to login

---

## Forgot Password Page

Route: `/forgot-password`

```
+------------------------------------------------------------------+
|                                                                  |
|                         [HarvestIQ Logo]                         |
|                                                                  |
|                    +-------------------------+                   |
|                    |    Reset Password       |                   |
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | Enter your email and    |                   |
|                    | we'll send a reset link |                   |
|                    |                         |                   |
|                    | Email                   |                   |
|                    | [___________________]   |                   |
|                    |                         |                   |
|                    | [   Send Reset Link   ] |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
|                                                                  |
|                    [Back to Sign In]                             |
|                                                                  |
+------------------------------------------------------------------+
```

### API Needed

```
POST /auth/forgot-password
  body: { email: string }
  response: { message: "If account exists, reset email sent" }
```

---

## Reset Password Page

Route: `/reset-password?token=X`

```
+------------------------------------------------------------------+
|                                                                  |
|                         [HarvestIQ Logo]                         |
|                                                                  |
|                    +-------------------------+                   |
|                    |    Set New Password     |                   |
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | New Password            |                   |
|                    | [___________________]   |                   |
|                    |                         |                   |
|                    | Confirm Password        |                   |
|                    | [___________________]   |                   |
|                    |                         |                   |
|                    | [   Reset Password    ] |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
|                                                                  |
+------------------------------------------------------------------+
```

### API Needed

```
POST /auth/reset-password
  body: { token: string, password: string }
  response: { message: "Password reset successful" }
```

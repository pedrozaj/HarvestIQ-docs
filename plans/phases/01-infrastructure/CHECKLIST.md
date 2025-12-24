# Phase 1: Implementation Checklist

Step-by-step implementation tasks for infrastructure and authentication.

---

## Backend Setup

### Environment & Configuration

- [ ] Create `HarvestIQ-backend/` directory structure per CODING-STANDARDS.md
- [ ] Initialize npm project with TypeScript
- [ ] Install dependencies:
  - [ ] express@5
  - [ ] pg (PostgreSQL client)
  - [ ] argon2 (password hashing)
  - [ ] jsonwebtoken (JWT)
  - [ ] zod (validation)
  - [ ] cors, helmet, morgan (middleware)
  - [ ] express-rate-limit (rate limiting)
  - [ ] cookie-parser (cookies)
  - [ ] nodemailer (email)
  - [ ] dotenv (environment)
- [ ] Create `src/config/env.ts` with Zod validation for env vars
- [ ] Create `src/config/database.ts` with connection pool
- [ ] Test database connection to Railway PostgreSQL

### Database Migrations

- [ ] Set up node-pg-migrate or similar migration tool
- [ ] Create migration: `001_create_builders.sql`
- [ ] Create migration: `002_create_users.sql`
- [ ] Create migration: `003_create_organizations.sql`
- [ ] Create migration: `004_create_organization_members.sql`
- [ ] Create migration: `005_create_refresh_tokens.sql`
- [ ] Run migrations against Railway database
- [ ] Verify tables created with correct constraints
- [ ] Verify indexes created for performance

### Constants & Types

- [ ] Create `src/constants/index.ts` per shared/CONSTANTS.md
- [ ] Create `src/types/index.ts` with TypeScript interfaces
- [ ] Export all constants and types

### Express App Setup

- [ ] Create `src/app.ts` with Express 5 setup
- [ ] Configure CORS for frontend domain
- [ ] Configure helmet for security headers
- [ ] Configure morgan for request logging
- [ ] Set up JSON body parsing
- [ ] Set up cookie parsing (cookie-parser)
- [ ] Add health check endpoint: `GET /health`

### Middleware

- [ ] Create `src/middleware/errorHandler.ts`
  - [ ] Handle AppError class
  - [ ] Handle ZodError
  - [ ] Handle generic errors
  - [ ] Log errors appropriately
- [ ] Create `src/middleware/validate.ts`
  - [ ] Accept Zod schema
  - [ ] Validate body, query, or params
  - [ ] Return standardized error format
- [ ] Create `src/middleware/auth.ts`
  - [ ] `requireAuth` - Verify JWT, attach user to request
  - [ ] `optionalAuth` - Attach user if token present
  - [ ] Extract token from cookie or Authorization header
  - [ ] Check subscription status on token refresh
- [ ] Create `src/middleware/rateLimiter.ts`
  - [ ] Auth endpoints: 10 requests/minute per IP
  - [ ] Register endpoint: 5 requests/minute per IP
  - [ ] Forgot password: 3 requests/minute per email
  - [ ] General endpoints: 100 requests/minute per user
  - [ ] Include rate limit headers in response

### Utilities

- [ ] Create `src/utils/crypto.ts`
  - [ ] `hashPassword(password)` - Argon2id hash
  - [ ] `verifyPassword(password, hash)` - Verify hash
  - [ ] `generateToken(length)` - Random token for reset/verification
  - [ ] `hashToken(token)` - SHA-256 for token storage
- [ ] Create `src/utils/jwt.ts`
  - [ ] `generateAccessToken(user)` - 15 min expiry
  - [ ] `generateRefreshToken(user)` - 7 day expiry with jti
  - [ ] `verifyToken(token)` - Verify and decode
- [ ] Create `src/utils/dates.ts` per shared/TIMESTAMPS.md

### Validation Schemas

- [ ] Create `src/schemas/auth.schema.ts`
  - [ ] loginSchema
  - [ ] registerSchema (company_name, name, email, password)
  - [ ] verifyEmailSchema
  - [ ] resendVerificationSchema
  - [ ] forgotPasswordSchema
  - [ ] resetPasswordSchema
  - [ ] changePasswordSchema

### Models

- [ ] Create `src/models/builder.model.ts`
  - [ ] `create(name)` - Create builder with trial status
  - [ ] `findById(id)` - Get builder
  - [ ] `update(id, data)` - Update builder settings
- [ ] Create `src/models/user.model.ts`
  - [ ] `create(builderId, data)` - Create user
  - [ ] `findByEmail(email)` - Find by email
  - [ ] `findById(id, builderId)` - Find by ID
  - [ ] `update(id, builderId, data)` - Update user
  - [ ] `verifyEmail(token)` - Verify email address
  - [ ] `setPasswordResetToken(email)` - Set reset token
  - [ ] `resetPassword(token, newPassword)` - Reset password
  - [ ] `updatePassword(id, builderId, newPassword)` - Change password
  - [ ] `incrementFailedLogins(email)` - Track failed attempts
  - [ ] `lockAccount(email)` - Lock after 5 failures
  - [ ] `resetFailedLogins(email)` - Reset on successful login
- [ ] Create `src/models/organization.model.ts`
  - [ ] `create(builderId, name, isDefault)` - Create org
  - [ ] `findByBuilder(builderId)` - List orgs
- [ ] Create `src/models/organizationMember.model.ts`
  - [ ] `create(builderId, orgId, userId, role)` - Add member
  - [ ] `findByUser(builderId, userId)` - Get user's memberships
- [ ] Create `src/models/refreshToken.model.ts`
  - [ ] `create(builderId, userId, tokenHash, deviceInfo)` - Create token
  - [ ] `findByTokenHash(hash)` - Find token
  - [ ] `revoke(id)` - Revoke single token
  - [ ] `revokeAllForUser(userId, exceptId)` - Revoke all except current
  - [ ] `listByUser(userId)` - List active sessions
  - [ ] `deleteExpired()` - Cleanup expired tokens

### Controllers

- [ ] Create `src/controllers/auth.controller.ts`
  - [ ] `register` - Create builder, user, org, send verification email
  - [ ] `verifyEmail` - Verify email with token
  - [ ] `resendVerification` - Resend verification email
  - [ ] `login` - Verify credentials, check email verified, check lockout, return tokens
  - [ ] `logout` - Revoke refresh token, clear cookies
  - [ ] `refresh` - Check subscription status, rotate token, return new tokens
  - [ ] `forgotPassword` - Send reset email
  - [ ] `resetPassword` - Reset with token, invalidate all sessions
- [ ] Create `src/controllers/user.controller.ts`
  - [ ] `getMe` - Get current user with builder and organizations
  - [ ] `updateMe` - Update profile (name, phone, timezone)
  - [ ] `changePassword` - Change password, invalidate other sessions
  - [ ] `updateNotificationPreferences` - Update notification settings
  - [ ] `listSessions` - List active sessions
  - [ ] `revokeSession` - Revoke specific session
  - [ ] `revokeAllSessions` - Revoke all except current
- [ ] Create `src/controllers/builder.controller.ts`
  - [ ] `get` - Get builder details
  - [ ] `update` - Update builder (admin only)

### Routes

- [ ] Create `src/routes/auth.routes.ts`
  - [ ] POST /auth/register (rate limited)
  - [ ] POST /auth/verify-email
  - [ ] POST /auth/resend-verification (rate limited)
  - [ ] POST /auth/login (rate limited)
  - [ ] POST /auth/logout
  - [ ] POST /auth/refresh
  - [ ] POST /auth/forgot-password (rate limited)
  - [ ] POST /auth/reset-password
- [ ] Create `src/routes/user.routes.ts`
  - [ ] GET /users/me (requireAuth)
  - [ ] PUT /users/me (requireAuth)
  - [ ] PUT /users/me/password (requireAuth)
  - [ ] PUT /users/me/notification-preferences (requireAuth)
  - [ ] GET /users/me/sessions (requireAuth)
  - [ ] DELETE /users/me/sessions/:id (requireAuth)
  - [ ] DELETE /users/me/sessions (requireAuth)
- [ ] Create `src/routes/builder.routes.ts`
  - [ ] GET /builder (requireAuth)
  - [ ] PUT /builder (requireAuth, admin)
- [ ] Create `src/routes/index.ts` - Aggregate all routes

### Email Service

- [ ] Create `src/services/email.service.ts`
  - [ ] Configure email provider (Resend/SendGrid)
  - [ ] `sendVerificationEmail(email, token)` - Send verification email
  - [ ] `sendPasswordResetEmail(email, token)` - Send reset email
- [ ] Create email verification email template
- [ ] Create password reset email template

### Security Testing

- [ ] Test account lockout after 5 failed attempts
- [ ] Test lockout expires after 15 minutes
- [ ] Test rate limiting on auth endpoints
- [ ] Test email verification is required before login
- [ ] Test subscription status check on token refresh
- [ ] Test password change invalidates other sessions
- [ ] Test refresh token rotation

---

## Frontend Setup

### Project Initialization

- [ ] Create Next.js 15 project in `HarvestIQ/`
- [ ] Configure TypeScript
- [ ] Install Tailwind CSS
- [ ] Install shadcn/ui and configure
- [ ] Install dependencies:
  - [ ] react-hook-form
  - [ ] @hookform/resolvers
  - [ ] zod
  - [ ] zustand (state management)
  - [ ] date-fns

### Configuration

- [ ] Set up environment variables
- [ ] Configure `next.config.js`
- [ ] Set up `tailwind.config.ts` per shared/THEMING.md
- [ ] Create `globals.css` with CSS variables

### Directory Structure

- [ ] Create directory structure per CODING-STANDARDS.md
- [ ] Set up path aliases in `tsconfig.json`

### Shared Components

- [ ] Install shadcn/ui components:
  - [ ] button
  - [ ] input
  - [ ] form
  - [ ] card
  - [ ] toast
  - [ ] dialog
  - [ ] alert
- [ ] Create Logo component

### API Client

- [ ] Create `src/lib/api.ts`
  - [ ] Base fetch wrapper with credentials: 'include'
  - [ ] Automatic token refresh on 401
  - [ ] Error handling for all error codes
  - [ ] Rate limit error handling (429)
  - [ ] Request/response interceptors

### Auth State

- [ ] Create `src/stores/authStore.ts`
  - [ ] Zustand store with persist
  - [ ] User state
  - [ ] isAuthenticated state
  - [ ] setUser function
  - [ ] clearUser function
- [ ] Create `src/hooks/useAuth.ts`
  - [ ] login function
  - [ ] register function
  - [ ] logout function
  - [ ] refreshUser function

### Validation Schemas

- [ ] Create `src/schemas/auth.schema.ts` (shared with backend)
  - [ ] loginSchema
  - [ ] registerSchema
  - [ ] forgotPasswordSchema
  - [ ] resetPasswordSchema

### Auth Layout

- [ ] Create `src/app/(auth)/layout.tsx`
- [ ] Style centered card layout
- [ ] Add logo

### Auth Pages

- [ ] Create `src/app/(auth)/login/page.tsx`
  - [ ] Form with email, password
  - [ ] Submit to POST /auth/login
  - [ ] Handle loading state
  - [ ] Handle "email not verified" error with resend link
  - [ ] Handle account locked error
  - [ ] Handle rate limit error
  - [ ] Redirect on success
- [ ] Create `src/app/(auth)/register/page.tsx`
  - [ ] Form with company name, name, email, password, confirm password
  - [ ] Password requirements display
  - [ ] Submit to POST /auth/register
  - [ ] Handle loading state
  - [ ] Handle errors
  - [ ] Show "check your email" message on success
- [ ] Create `src/app/(auth)/verify-email/page.tsx`
  - [ ] Extract token from URL query params
  - [ ] Auto-submit to POST /auth/verify-email
  - [ ] Show success message
  - [ ] Link to login
  - [ ] Handle invalid/expired token
- [ ] Create `src/app/(auth)/forgot-password/page.tsx`
  - [ ] Form with email
  - [ ] Submit to POST /auth/forgot-password
  - [ ] Show success message (always, for security)
- [ ] Create `src/app/(auth)/reset-password/page.tsx`
  - [ ] Extract token from URL query params
  - [ ] Form with new password, confirm password
  - [ ] Submit to POST /auth/reset-password
  - [ ] Handle invalid/expired token
  - [ ] Redirect to login on success

### Protected Routes

- [ ] Create `src/middleware.ts`
  - [ ] Check for auth token/session
  - [ ] Redirect to login if missing
  - [ ] Redirect to dashboard if already logged in (on auth pages)

### Dashboard Shell

- [ ] Create `src/app/(dashboard)/layout.tsx` (empty shell for now)
- [ ] Create `src/app/(dashboard)/dashboard/page.tsx` (placeholder)
- [ ] Verify protected route redirect works

---

## Integration Testing

- [ ] Test full registration flow (frontend → backend → database)
- [ ] Test verification email sends correctly
- [ ] Test verification flow (click link → email verified)
- [ ] Test login blocked until email verified
- [ ] Test full login flow
- [ ] Test session persistence across page reload
- [ ] Test logout clears session
- [ ] Test password reset email sends
- [ ] Test password reset updates password
- [ ] Test password reset invalidates all sessions
- [ ] Test expired token redirect to login
- [ ] Test account lockout after 5 failed attempts
- [ ] Test rate limiting displays appropriate error

---

## Deployment

- [ ] Deploy backend to Railway
- [ ] Configure environment variables on Railway
- [ ] Run migrations on Railway database
- [ ] Verify email service works in production
- [ ] Deploy frontend to Vercel
- [ ] Configure environment variables on Vercel
- [ ] Test production authentication flow
- [ ] Test CORS configuration
- [ ] Verify cookies set correctly in production

---

## Definition of Done

### Authentication
- [ ] User can register new account
- [ ] Verification email sends on registration
- [ ] User cannot login until email verified
- [ ] User can login with verified email/password
- [ ] Account locks after 5 failed login attempts
- [ ] JWT stored in httpOnly cookie
- [ ] Token refresh extends session
- [ ] Token rotation on refresh
- [ ] User can logout (tokens revoked)
- [ ] Password reset flow works end-to-end
- [ ] Password change invalidates other sessions

### Security
- [ ] Rate limiting active on auth endpoints
- [ ] Subscription status checked on token refresh
- [ ] All endpoints require HTTPS in production
- [ ] CORS restricts to allowed origins

### User Management
- [ ] User can view profile
- [ ] User can update name, phone, timezone
- [ ] User can change password
- [ ] User can view active sessions
- [ ] User can revoke sessions

### Multi-Tenancy
- [ ] All queries filter by builder_id
- [ ] User cannot access other builders' data

### Frontend
- [ ] Protected routes require authentication
- [ ] Frontend redirects appropriately
- [ ] Error messages display correctly
- [ ] No console errors in browser
- [ ] API returns proper error formats

### Database
- [ ] All migrations run successfully
- [ ] Indexes created for performance
- [ ] Soft delete pattern implemented

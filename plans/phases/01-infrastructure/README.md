# Phase 1: Infrastructure & Authentication

Foundation layer for the entire application. Must be completed before any other phase.

---

## Phase Goal

Establish the technical foundation: database, authentication, authorization, and API infrastructure.

---

## Dependencies

- None (this is the foundation phase)

---

## Deliverables

1. PostgreSQL database running on Railway with connection pooling
2. User registration with email verification flow
3. Login with JWT authentication and account lockout protection
4. Multi-tenant data isolation via `builder_id`
5. Protected API routes with role-based access
6. Rate limiting on auth endpoints
7. Frontend auth pages and protected route handling

---

## Tables Introduced

| Table | Purpose |
|-------|---------|
| `builders` | Tenant/company accounts |
| `users` | Individual user accounts |
| `organizations` | Business units within a builder |
| `organization_members` | User-organization relationships |
| `refresh_tokens` | Session tracking and revocation |

---

## Endpoints Introduced

### Authentication (no auth required)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/auth/register` | New builder registration |
| POST | `/auth/verify-email` | Verify email with token |
| POST | `/auth/resend-verification` | Resend verification email |
| POST | `/auth/login` | User login |
| POST | `/auth/logout` | User logout |
| POST | `/auth/refresh` | Refresh JWT token |
| POST | `/auth/forgot-password` | Initiate password reset |
| POST | `/auth/reset-password` | Complete password reset |

### User (auth required)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/users/me` | Get current user |
| PUT | `/users/me` | Update current user |
| PUT | `/users/me/password` | Change password |
| PUT | `/users/me/notification-preferences` | Update notifications |
| GET | `/users/me/sessions` | List active sessions |
| DELETE | `/users/me/sessions/:id` | Revoke session |
| DELETE | `/users/me/sessions` | Revoke all other sessions |

### Builder (auth required)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/builder` | Get builder details |
| PUT | `/builder` | Update builder (admin only) |

---

## UI Pages Introduced

| Route | Page | Purpose |
|-------|------|---------|
| `/login` | Login | User authentication |
| `/register` | Register | New builder signup |
| `/verify-email` | Verify Email | Email verification |
| `/forgot-password` | Forgot Password | Password reset request |
| `/reset-password` | Reset Password | Password reset form |

---

## Technical Decisions

| Component | Choice | Rationale |
|-----------|--------|-----------|
| Database | PostgreSQL on Railway | Managed service, pgvector support for AI phase |
| Auth | JWT with refresh tokens | Stateless, scalable, httpOnly cookies for security |
| Password hashing | Argon2id | Memory-hard, OWASP recommended, no length limit |
| Token rotation | Refresh token rotation | New token on each refresh for security |
| Rate limiting | express-rate-limit | In-memory store for MVP |
| Validation | Zod | TypeScript-first, works on frontend and backend |
| API framework | Express 5 | Async error handling, mature ecosystem |

---

## Security Features

| Feature | Implementation |
|---------|----------------|
| Password hashing | Argon2id |
| Token storage | httpOnly cookies, Secure, SameSite=Strict |
| Refresh tokens | Hashed (SHA-256) before database storage |
| Account lockout | 5 failed attempts → 15 minute lockout |
| Rate limiting | Auth endpoints: 5-10/min, General: 100/min |
| Session invalidation | Password change revokes all other sessions |
| Subscription check | Token refresh validates subscription status |

---

## Acceptance Criteria

### Authentication

- [ ] Database migrations run successfully
- [ ] User can register with company name, name, email, password
- [ ] Registration sends verification email
- [ ] User cannot login until email is verified
- [ ] User can resend verification email
- [ ] User can login with verified email and password
- [ ] Failed login increments counter; 5 failures locks account for 15 min
- [ ] JWT token stored in httpOnly cookie
- [ ] Access token expires after 15 minutes
- [ ] Refresh token rotation works correctly
- [ ] User can logout (tokens revoked)
- [ ] Password reset email sends successfully
- [ ] User can set new password with reset token
- [ ] Password change invalidates all other sessions

### Authorization

- [ ] Protected routes return 401 for unauthenticated requests
- [ ] Protected routes return 403 for unauthorized requests
- [ ] Token refresh checks subscription status

### User Management

- [ ] User can view their profile
- [ ] User can update name, phone, timezone
- [ ] User can change password (requires current password)
- [ ] User can view active sessions
- [ ] User can revoke individual sessions
- [ ] User can revoke all other sessions

### Multi-Tenancy

- [ ] All queries filter by builder_id
- [ ] User cannot access data from other builders

### Frontend

- [ ] Frontend redirects to login when unauthenticated
- [ ] Frontend handles token refresh automatically
- [ ] Frontend shows appropriate error messages
- [ ] Rate limiting errors are handled gracefully

---

## Files to Create

### Backend

```
src/
├── config/
│   ├── database.ts          # PostgreSQL connection
│   └── env.ts               # Environment validation
├── constants/
│   └── index.ts             # Shared constants
├── controllers/
│   ├── auth.controller.ts   # Auth route handlers
│   ├── user.controller.ts   # User route handlers
│   └── builder.controller.ts # Builder route handlers
├── middleware/
│   ├── auth.ts              # JWT verification
│   ├── errorHandler.ts      # Global error handling
│   ├── rateLimiter.ts       # Rate limiting
│   └── validate.ts          # Request validation
├── models/
│   ├── user.model.ts        # User queries
│   ├── builder.model.ts     # Builder queries
│   ├── organization.model.ts # Organization queries
│   └── refreshToken.model.ts # Token queries
├── routes/
│   ├── index.ts             # Route aggregator
│   ├── auth.routes.ts       # Auth endpoints
│   ├── user.routes.ts       # User endpoints
│   └── builder.routes.ts    # Builder endpoints
├── schemas/
│   ├── auth.schema.ts       # Auth validation schemas
│   └── user.schema.ts       # User validation schemas
├── services/
│   └── email.service.ts     # Email sending
├── utils/
│   ├── crypto.ts            # Password hashing
│   └── jwt.ts               # Token utilities
├── types/
│   └── index.ts             # TypeScript types
└── app.ts                   # Express setup
```

### Frontend

```
src/
├── app/
│   ├── (auth)/
│   │   ├── layout.tsx       # Auth layout
│   │   ├── login/page.tsx
│   │   ├── register/page.tsx
│   │   ├── verify-email/page.tsx
│   │   ├── forgot-password/page.tsx
│   │   └── reset-password/page.tsx
│   └── layout.tsx           # Root layout
├── components/
│   ├── ui/                  # shadcn components
│   └── forms/
│       ├── LoginForm.tsx
│       ├── RegisterForm.tsx
│       └── PasswordResetForm.tsx
├── hooks/
│   └── useAuth.ts           # Auth hook
├── lib/
│   ├── api.ts               # API client
│   └── constants.ts         # Shared constants
├── stores/
│   └── authStore.ts         # Auth state (Zustand)
├── schemas/
│   └── auth.schema.ts       # Validation schemas
└── middleware.ts            # Next.js middleware
```

### Migrations

```
migrations/
├── 001_create_builders.sql
├── 002_create_users.sql
├── 003_create_organizations.sql
├── 004_create_organization_members.sql
└── 005_create_refresh_tokens.sql
```

---

## Out of Scope

These items are handled in later phases:

- Team invitations (Phase 2: Organizations & Teams)
- Organization CRUD (Phase 2: Organizations & Teams)
- File storage / R2 setup (Phase 4: Documents)
- Background job queue (Phase 7: Notifications)
- Activity logging (Phase 7: Notifications)

---

## Related Documentation

- Schema details: [SCHEMA.md](./SCHEMA.md)
- API endpoints: [API.md](./API.md)
- UI wireframes: [UI.md](./UI.md)
- Implementation checklist: [CHECKLIST.md](./CHECKLIST.md)

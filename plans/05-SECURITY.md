# Security Architecture

Comprehensive security plan for HarvestIQ infrastructure.

---

## Authentication & Authorization

### User Authentication
- Password requirements: minimum 8 characters
- Password hashing: Argon2id
- Account lockout after 5 failed attempts (15 minute lockout)
- Email verification required before full access
- Session management via JWT tokens
- Session invalidation on password change

### Token Strategy
```
Access Token (JWT):
- Short-lived: 15 minutes
- Payload contains:
  - user_id
  - builder_id (tenant isolation)
  - email
  - iat, exp
- Stored: Memory only (not localStorage)
- Signed with RS256 or HS256

Refresh Token:
- Long-lived: 7 days
- Stored: HTTP-only, Secure, SameSite=Strict cookie
- Hashed (SHA-256) before storage in database
- Rotated on each use (old token invalidated)
- Device tracking for security audit
```

### Token Refresh Flow
```
1. Access token expires
2. Client sends request, receives 401
3. Client calls /auth/refresh (refresh token sent via cookie)
4. Server validates:
   - Refresh token exists and not revoked
   - Token not expired
   - User account is active
   - Builder subscription is active (not cancelled/past_due)
5. Server issues new access token
6. Server rotates refresh token (new token, old invalidated)
7. Client retries original request
```

### Role Enforcement
- All API endpoints validate user role
- Admin: full organization access, can manage users
- Member: only assigned project access
- Middleware checks on every request
- Roles stored in organization_members table

---

## Multi-Tenant Isolation

### Builder-Level Isolation
- Every table includes `builder_id` column
- All queries filtered by builder_id from JWT
- Middleware injects builder context on every request
- Database-level row security (optional, for defense in depth)

### Query Filtering Pattern
```javascript
// Middleware extracts from JWT
req.builder_id = jwt.builder_id;

// Every query includes builder filter
const projects = await db.query(
  'SELECT * FROM projects WHERE builder_id = $1 AND ...',
  [req.builder_id]
);
```

### Cross-Tenant Prevention
- Foreign key references validated within same builder
- JOIN operations always include builder_id match
- API responses never include builder_id (internal only)
- UUIDs prevent enumeration attacks

---

## API Security

### Request Validation
- All inputs validated and sanitized
- Parameterized queries (prevents SQL injection)
- Request body size limits (1MB default, 50MB for uploads)
- Content-type validation
- Schema validation on all inputs (using Zod or similar)

### Rate Limiting
```
Limits per user:
- General API: 100 requests/minute
- Auth endpoints: 10 requests/minute per IP
- File upload: 10 requests/minute
- AI queries: 20 requests/minute

Limits per builder:
- Total API: 1000 requests/minute
- File upload: 100 requests/minute
- AI queries: 100 requests/minute
```

### CORS Configuration
```javascript
const corsOptions = {
  origin: [
    'https://harvest-iq-rosy.vercel.app',
    process.env.NODE_ENV === 'development' && 'http://localhost:3000'
  ].filter(Boolean),
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Authorization', 'Content-Type'],
  credentials: true,
  maxAge: 86400
};
```

### Security Headers
```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

---

## Service-to-Service Communication

### Vercel to Railway (Frontend to Backend API)
```
Authentication: JWT Bearer token (user's access token)
Transport: HTTPS only (TLS 1.2+)
Validation: Backend validates token on every request
Builder context: Extracted from validated JWT
```

### Railway to Railway (Backend to Database)
```
Connection: Private networking within Railway project
Authentication: Database credentials (connection string)
Encryption: SSL/TLS required (sslmode=require)
Connection pooling: Built-in or PgBouncer
Connection limit: Configured per environment
```

### Railway to Cloudflare R2 (Backend to Storage)
```
Authentication: AWS-style access key + secret key
Transport: HTTPS only
Credentials: Environment variables, never in code
Operations: All file operations proxied through backend API
Bucket: Private, no public access
```

---

## Document Security

### Upload Security
- File type whitelist: PDF, PNG, JPG, JPEG, GIF, WEBP, DOC, DOCX, XLS, XLSX
- File size limit: 50MB per file
- MIME type validation (check magic bytes, not just extension)
- Filename sanitization (remove path traversal, special chars)
- Store with random UUID keys, not original filenames
- Virus/malware scanning (via ClamAV or external service)

### Storage Quota
- Per-builder storage limits
- Track usage in builders.storage_used_bytes
- Enforce limits on upload
- Cleanup deleted files from R2

### Access Control
- All document access through authenticated API endpoints
- Verify user has access to parent project
- Verify project belongs to user's builder
- No direct R2 bucket access from frontend

### Download Security
```
1. User requests document download
2. Backend validates:
   - User is authenticated
   - User's builder matches document's builder
   - User has access to project (admin or member with project access)
   - Document exists and not soft-deleted
3. Backend generates signed R2 URL:
   - Expiry: 15 minutes
   - Scoped to specific object key
4. Backend returns signed URL to frontend
5. Frontend redirects/fetches from signed URL
6. Log document access to activity_log
```

---

## Data Protection

### Data at Rest
- Database: Encrypted at rest (Railway managed)
- R2 Storage: Encrypted at rest (Cloudflare managed)
- Backups: Encrypted

### Data in Transit
- All connections use TLS 1.2+
- HTTPS enforced on all endpoints
- HTTP requests rejected (301 redirect to HTTPS)

### Sensitive Data Handling
```
Hashed (one-way):
- Passwords (Argon2id)
- Refresh tokens (SHA-256)
- Invitation tokens (SHA-256)

Never logged:
- Passwords
- Tokens
- Full credit card numbers
- SSN or similar PII

Never returned in API responses:
- password_hash
- token_hash
- builder_id (internal use only)
```

### Data Isolation
- Multi-tenant data isolation via builder_id at top level
- All queries filtered by builder context first, then organization
- No cross-builder data leakage possible
- No cross-organization data leakage within a builder
- Soft deletes preserve data integrity

---

## Environment Security

### Secret Management
```
Production secrets stored in:
- Railway environment variables (backend)
- Vercel environment variables (frontend)

Required secrets:
- DATABASE_URL (connection string with SSL)
- JWT_SECRET (256-bit minimum, for signing)
- JWT_REFRESH_SECRET (256-bit minimum)
- R2_ACCESS_KEY_ID
- R2_SECRET_ACCESS_KEY
- R2_BUCKET_NAME
- R2_ENDPOINT
- SENDGRID_API_KEY (email)
- TWILIO_ACCOUNT_SID (SMS)
- TWILIO_AUTH_TOKEN (SMS)
- TWILIO_PHONE_NUMBER (SMS)
- CLAUDE_API_KEY (AI)
- OPENAI_API_KEY (embeddings)
```

### Environment Separation
- Separate databases for development/staging/production
- Separate R2 buckets per environment
- No production credentials in development
- Environment-specific CORS origins

---

## Audit & Monitoring

### Audit Logging
All security-relevant events logged to activity_log table:
```
Events to log:
- Authentication (login, logout, failed attempts)
- Authorization failures (403 responses)
- Document access (view, download)
- Data modifications (create, update, delete)
- Admin actions (user management, role changes)
- Subscription changes
- Password changes
- Token revocations

Log fields:
- builder_id
- user_id
- action
- entity_type
- entity_id
- ip_address
- user_agent
- created_at
- changes (JSONB for before/after on updates)
```

### Security Monitoring
- Failed authentication spike detection
- Rate limit breach alerts
- Error rate monitoring
- Unusual access pattern detection
- Multiple failed attempts from same IP
- Access from new geographic locations (optional)

### Log Retention
- Security logs: 90 days minimum
- Activity logs: 1 year
- Access logs: 30 days
- Audit logs: As required by compliance

---

## Security Checklist

### Pre-Launch
- [ ] All endpoints require authentication (except public routes)
- [ ] Role-based access enforced on all data operations
- [ ] Builder-level isolation verified on all queries
- [ ] SQL injection protection verified (parameterized queries)
- [ ] XSS protection verified (input sanitization, CSP headers)
- [ ] CSRF protection implemented (SameSite cookies)
- [ ] Rate limiting active
- [ ] CORS properly configured (no wildcards in production)
- [ ] Security headers in place
- [ ] Secrets not in codebase
- [ ] HTTPS enforced
- [ ] Document access properly gated
- [ ] Signed URLs have appropriate expiry
- [ ] Audit logging active
- [ ] Error messages don't leak sensitive info
- [ ] JWT includes builder_id for tenant isolation
- [ ] Subscription status checked on token refresh

### Ongoing
- [ ] Dependency vulnerability scanning (npm audit, Snyk)
- [ ] Regular security updates
- [ ] Periodic access review
- [ ] Log review and monitoring
- [ ] Penetration testing (annual)
- [ ] Incident response plan documented and tested

---

## Incident Response

### Suspected Breach Protocol
1. Immediately revoke all refresh tokens (clear refresh_tokens table)
2. Force password reset for affected users
3. Rotate all API secrets
4. Review audit logs for scope of breach
5. Notify affected users if data exposure confirmed
6. Document incident and remediation
7. Post-mortem and security improvements

### Token Compromise
1. Revoke specific user's refresh tokens
2. Force re-authentication
3. Review user's recent activity in activity_log
4. Notify user of security event
5. If widespread, trigger full token rotation

### Data Breach Notification
- Internal notification within 1 hour
- User notification within 72 hours if PII exposed
- Regulatory notification as required
- Public disclosure if significant

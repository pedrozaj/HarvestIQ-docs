# Phase 2: Core Data Models

Establish project and organization management capabilities.

---

## Phase Goal

Enable users to create and manage projects, organizations, and team members.

---

## Dependencies

- Phase 1: Infrastructure & Authentication (complete)

---

## Deliverables

1. Organization management with member invitations
2. Project CRUD with multi-unit development tracking
3. Activity logging for audit trail
4. App shell with navigation sidebar
5. Team management page for admins

---

## Tables Introduced

| Table | Purpose |
|-------|---------|
| `projects` | Construction project records |
| `activity_log` | Audit trail for all actions |

---

## Endpoints Introduced

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/organizations` | List user's organizations |
| GET | `/organizations/:id` | Get organization details |
| PUT | `/organizations/:id` | Update organization |
| GET | `/organizations/:id/members` | List members |
| POST | `/organizations/:id/members/invite` | Send invitation |
| PUT | `/organizations/:id/members/:userId/role` | Change role |
| DELETE | `/organizations/:id/members/:userId` | Remove member |
| GET | `/invitations/validate` | Validate invitation token |
| POST | `/organizations/:id/members/accept` | Accept invitation |
| GET | `/projects` | List projects |
| POST | `/projects` | Create project |
| GET | `/projects/:id` | Get project details |
| PUT | `/projects/:id` | Update project |
| DELETE | `/projects/:id` | Archive project |
| GET | `/projects/:id/summary` | Get calculated summary |
| GET | `/projects/:id/activity` | Get project activity |
| GET | `/activity` | Get builder-wide activity |

---

## UI Pages Introduced

| Route | Page | Purpose |
|-------|------|---------|
| `/dashboard` | Dashboard | Multi-project overview (shell) |
| `/projects` | Projects List | View and filter projects |
| `/projects/:id` | Project Detail | Project tabs (shell) |
| `/team` | Team Management | Manage members (admin) |

---

## Technical Decisions

| Component | Choice | Rationale |
|-----------|--------|-----------|
| Soft deletes | `deleted_at` column | Maintain data for reporting |
| Activity log | Append-only table | Immutable audit trail |
| Invitations | Token-based | Email link acceptance |

---

## Acceptance Criteria

- [ ] Admin can invite team members via email
- [ ] Invited users can accept and join organization
- [ ] Admin can change member roles
- [ ] Admin can remove members
- [ ] User can create new projects
- [ ] User can view project list with filters
- [ ] User can update project details
- [ ] User can archive projects
- [ ] All actions logged to activity_log
- [ ] Sidebar navigation works on all pages

---

## Related Documentation

- Schema details: [SCHEMA.md](./SCHEMA.md)
- API endpoints: [API.md](./API.md)
- UI wireframes: [UI.md](./UI.md)
- Implementation checklist: [CHECKLIST.md](./CHECKLIST.md)

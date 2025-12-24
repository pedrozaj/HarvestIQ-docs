# Phase 2: Implementation Checklist

Step-by-step tasks for core models implementation.

---

## Database

- [ ] Create migration: `005_create_projects.sql`
- [ ] Create migration: `006_create_activity_log.sql`
- [ ] Create migration: `007_create_invitations.sql` (optional)
- [ ] Run migrations
- [ ] Verify table constraints and indexes

---

## Backend Models

### Organization Model
- [ ] `findByUser(builderId, userId)` - Get user's organizations
- [ ] `findById(id, builderId)` - Get organization details
- [ ] `update(id, builderId, data)` - Update organization

### Organization Member Model
- [ ] `findByOrg(builderId, orgId)` - List members
- [ ] `updateRole(builderId, orgId, userId, role)` - Change role
- [ ] `remove(builderId, orgId, userId)` - Remove member
- [ ] `getAdminCount(builderId, orgId)` - Count admins

### Invitation Model
- [ ] `create(builderId, orgId, data)` - Create invitation
- [ ] `findByToken(token)` - Get by token
- [ ] `findPendingByOrg(builderId, orgId)` - List pending
- [ ] `accept(token, userId)` - Accept invitation
- [ ] `resend(id, builderId)` - Resend email
- [ ] `cancel(id, builderId)` - Cancel invitation

### Project Model
- [ ] `findByBuilder(builderId, filters)` - List with filters
- [ ] `findById(id, builderId)` - Get project
- [ ] `create(builderId, data)` - Create project
- [ ] `update(id, builderId, data)` - Update project
- [ ] `remove(id, builderId)` - Soft delete
- [ ] `getSummary(id, builderId)` - Calculate summary
- [ ] `compare(ids, builderId)` - Compare projects

### Activity Model
- [ ] `log(params)` - Create activity entry
- [ ] `findByProject(builderId, projectId)` - Project activity
- [ ] `findByBuilder(builderId, filters)` - Builder activity

---

## Backend Controllers

### Organization Controller
- [ ] `getAll` - GET /organizations
- [ ] `getById` - GET /organizations/:id
- [ ] `update` - PUT /organizations/:id

### Member Controller
- [ ] `getMembers` - GET /organizations/:id/members
- [ ] `invite` - POST /organizations/:id/members/invite
- [ ] `updateRole` - PUT /organizations/:id/members/:userId/role
- [ ] `remove` - DELETE /organizations/:id/members/:userId

### Invitation Controller
- [ ] `validate` - GET /invitations/validate
- [ ] `accept` - POST /organizations/:id/members/accept
- [ ] `resend` - POST /organizations/:id/invitations/:id/resend
- [ ] `cancel` - DELETE /organizations/:id/invitations/:id

### Project Controller
- [ ] `getAll` - GET /projects
- [ ] `create` - POST /projects
- [ ] `getById` - GET /projects/:id
- [ ] `update` - PUT /projects/:id
- [ ] `remove` - DELETE /projects/:id
- [ ] `getSummary` - GET /projects/:id/summary
- [ ] `compare` - GET /projects/compare

### Activity Controller
- [ ] `getProjectActivity` - GET /projects/:id/activity
- [ ] `getBuilderActivity` - GET /activity

---

## Backend Routes

- [ ] Create `src/routes/organization.routes.ts`
- [ ] Create `src/routes/project.routes.ts`
- [ ] Create `src/routes/activity.routes.ts`
- [ ] Register routes in `src/routes/index.ts`

---

## Middleware

- [ ] Create `requireOrgAdmin` - Check admin role for org
- [ ] Create `requireOrgMember` - Check membership in org
- [ ] Create `requireProjectAccess` - Check project access

---

## Validation Schemas

- [ ] `organizationUpdateSchema`
- [ ] `invitationCreateSchema`
- [ ] `memberRoleUpdateSchema`
- [ ] `projectCreateSchema`
- [ ] `projectUpdateSchema`
- [ ] `projectQuerySchema`

---

## Email Templates

- [ ] Create invitation email template
- [ ] Configure email service for invitations

---

## Frontend Components

### Layout
- [ ] Create `Sidebar.tsx` with navigation
- [ ] Create `Header.tsx` with user menu
- [ ] Create `UserMenu.tsx` dropdown
- [ ] Create `DashboardLayout.tsx`

### UI Components
- [ ] Install shadcn: table, dropdown-menu, select, tabs
- [ ] Create `StatusBadge.tsx`
- [ ] Create `DataTable.tsx` (reusable)
- [ ] Create `Pagination.tsx`
- [ ] Create `EmptyState.tsx`

### Projects
- [ ] Create `ProjectCard.tsx`
- [ ] Create `ProjectForm.tsx`
- [ ] Create `ProjectFilters.tsx`

### Team
- [ ] Create `MemberTable.tsx`
- [ ] Create `InviteForm.tsx`
- [ ] Create `RoleChangeDialog.tsx`
- [ ] Create `RemoveMemberDialog.tsx`

---

## Frontend Pages

### Dashboard Layout
- [ ] Create `src/app/(dashboard)/layout.tsx`
- [ ] Integrate Sidebar and Header
- [ ] Set up protected route

### Dashboard Page
- [ ] Create `src/app/(dashboard)/dashboard/page.tsx`
- [ ] Add project cards grid
- [ ] Add summary stats

### Projects List
- [ ] Create `src/app/(dashboard)/projects/page.tsx`
- [ ] Implement filters
- [ ] Implement search
- [ ] Implement pagination
- [ ] Add create modal

### Project Detail
- [ ] Create `src/app/(dashboard)/projects/[id]/page.tsx`
- [ ] Create tab navigation
- [ ] Create overview tab content
- [ ] Load project summary

### Team Page
- [ ] Create `src/app/(dashboard)/team/page.tsx`
- [ ] Members tab with table
- [ ] Invitations tab
- [ ] Invite modal

---

## Frontend Hooks

- [ ] `useProjects(filters)` - List projects
- [ ] `useProject(id)` - Single project
- [ ] `useCreateProject()` - Create mutation
- [ ] `useUpdateProject()` - Update mutation
- [ ] `useDeleteProject()` - Delete mutation
- [ ] `useOrganizations()` - List orgs
- [ ] `useOrgMembers(orgId)` - List members
- [ ] `useInviteMember()` - Invite mutation

---

## Integration Testing

- [ ] Test project CRUD flow
- [ ] Test project filtering and search
- [ ] Test invitation flow end-to-end
- [ ] Test role change flow
- [ ] Test member removal flow
- [ ] Test activity logging
- [ ] Test authorization (admin vs member)

---

## Definition of Done

- [ ] Admin can invite team members
- [ ] Invitation emails send correctly
- [ ] Users can accept invitations
- [ ] Admin can change roles
- [ ] Admin can remove members
- [ ] Cannot demote last admin
- [ ] Projects can be created
- [ ] Projects can be filtered/searched
- [ ] Projects can be updated
- [ ] Projects can be archived
- [ ] Project summary calculates correctly
- [ ] Activity logged for all actions
- [ ] Sidebar navigation works
- [ ] All pages load without errors

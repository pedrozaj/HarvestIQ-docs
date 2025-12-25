# Phase 6: Implementation Checklist

Organized into parallel work streams for efficient agent execution.

---

## Prerequisites

- [x] Phases 1-4 complete (migrations 001-019)
- [ ] Phase 5 complete (migrations 020-025, task_items, reminders, notifications)

---

## Work Stream A: Dashboard Backend

**Agent Context:** Creates the dashboard aggregation endpoint. Requires Phase 5 tables.

**Files to reference:**
- API contracts: `API.md`
- Existing controller pattern: `/src/controllers/project.controller.ts`

### A1: Dashboard Controller
- [ ] Create `src/controllers/dashboard.controller.ts`
- [ ] Implement aggregation queries:
  - [ ] Active project count (status = 'active')
  - [ ] Total budget/spent (SUM from budget_items)
  - [ ] Overdue tasks count (task_items where due_date < NOW())
  - [ ] Upcoming milestones count (next 30 days)
  - [ ] Unpaid invoices amount (invoices where status != 'paid')
- [ ] Get user's projects with metrics:
  - [ ] Budget progress percentage
  - [ ] Schedule progress percentage
  - [ ] Next milestone info
- [ ] Get user's task items (limit 5, assigned_to = user.id)
- [ ] Get user's upcoming reminders (limit 5)
- [ ] Get recent activity (limit 10)

### A2: Dashboard Routes
- [ ] Create `src/routes/dashboard.routes.ts`
- [ ] Register `GET /dashboard` endpoint
- [ ] Add to `src/routes/index.ts`

---

## Work Stream B: Search & Reports Backend

**Agent Context:** Creates search and report endpoints. Can run parallel to Stream A.

**Files to reference:**
- API contracts: `API.md`

### B1: Search Controller
- [ ] Create `src/controllers/search.controller.ts`
- [ ] Implement search queries using ILIKE or full-text search:
  - [ ] Projects: name, address
  - [ ] Schedule tasks: name
  - [ ] Documents: name
  - [ ] Invoices: invoice_number, description
  - [ ] Task items: title
- [ ] Limit results per type (default 5)
- [ ] Scope all queries by builder_id

### B2: Reports Controller
- [ ] Create `src/controllers/reports.controller.ts`
- [ ] Schedule report endpoint:
  - [ ] Task variance calculations (planned vs actual dates)
  - [ ] Milestone status summary
  - [ ] On-time completion percentage
- [ ] Budget report endpoint:
  - [ ] Category breakdown (SUM by category)
  - [ ] Phase breakdown (SUM by phase)
  - [ ] Monthly spending trend
- [ ] Cross-project spending endpoint:
  - [ ] By project totals
  - [ ] By category totals
  - [ ] Efficiency comparison (cost per unit)

### B3: Search & Reports Routes
- [ ] Create `src/routes/search.routes.ts`
- [ ] Create `src/routes/reports.routes.ts`
- [ ] Register endpoints:
  - [ ] `GET /search`
  - [ ] `GET /projects/:id/reports/schedule`
  - [ ] `GET /projects/:id/reports/budget`
  - [ ] `GET /reports/spending`
- [ ] Add to `src/routes/index.ts`

---

## Work Stream C: Frontend Components

**Agent Context:** Creates all frontend UI. Requires Streams A & B for API endpoints.

**Files to reference:**
- UI wireframes: `UI.md`
- API contracts: `API.md`
- Existing patterns: `/src/components/payments/`

### C1: Frontend Hooks
- [ ] Create `src/hooks/useDashboard.ts` - Fetch dashboard data
- [ ] Create `src/hooks/useSearch.ts` - Search with debounce
- [ ] Create `src/hooks/useScheduleReport.ts`
- [ ] Create `src/hooks/useBudgetReport.ts`
- [ ] Create `src/hooks/useSpendingReport.ts`

### C2: Dashboard Components
- [ ] Create `src/components/dashboard/SummaryCard.tsx`
- [ ] Create `src/components/dashboard/ProjectCard.tsx`
- [ ] Create `src/components/dashboard/TaskItemPreview.tsx`
- [ ] Create `src/components/dashboard/ReminderPreview.tsx`
- [ ] Create `src/components/dashboard/InsightCard.tsx`
- [ ] Create `src/components/dashboard/ActivityItem.tsx`
- [ ] Create `src/components/dashboard/ActivityFeed.tsx`

### C3: Search Components
- [ ] Create `src/components/search/SearchModal.tsx`
- [ ] Create `src/components/search/SearchInput.tsx`
- [ ] Create `src/components/search/SearchResults.tsx`
- [ ] Create `src/components/search/SearchResultItem.tsx`
- [ ] Implement keyboard navigation (↑↓ Enter Esc)
- [ ] Add Cmd+K / Ctrl+K global shortcut

### C4: Settings Components
- [ ] Create `src/components/settings/ProfileForm.tsx`
- [ ] Create `src/components/settings/PasswordForm.tsx`
- [ ] Create `src/components/settings/SettingsTabs.tsx`

### C5: Pages
- [ ] Update `src/app/dashboard/page.tsx` - Full dashboard
- [ ] Create `src/app/settings/page.tsx` - Profile settings
- [ ] Create `src/app/settings/security/page.tsx` - Password change
- [ ] Verify notifications settings from Phase 5 works

### C6: Layout Updates
- [ ] Add SearchModal trigger to header
- [ ] Add Cmd+K event listener to app
- [ ] Update sidebar with Settings link

---

## Integration & Testing

**Agent Context:** Run after all streams complete.

### Testing Checklist
- [ ] Dashboard loads with correct aggregated data
- [ ] Summary cards show accurate counts
- [ ] Project cards display progress percentages
- [ ] Task items and reminders preview shows
- [ ] Search returns relevant results
- [ ] Search respects builder isolation
- [ ] Keyboard shortcuts work (Cmd+K)
- [ ] Reports calculate correctly
- [ ] Profile settings save correctly
- [ ] Password change works

---

## Audit Checklist

**Run after all implementation is complete:**

- [ ] All API endpoints return expected responses
- [ ] Frontend compiles without TypeScript errors
- [ ] Backend compiles without TypeScript errors
- [ ] Dashboard data matches database queries
- [ ] Search results are accurate
- [ ] Reports show correct calculations
- [ ] No console errors in browser
- [ ] Keyboard shortcuts work on Mac and Windows

---

## Definition of Done

- [ ] Dashboard shows multi-project overview with accurate stats
- [ ] Project cards show budget/schedule progress
- [ ] Task items and reminders display on dashboard
- [ ] AI insights show with dismiss functionality
- [ ] Activity feed shows recent changes
- [ ] Global search works across all entity types
- [ ] Cmd+K / Ctrl+K opens search modal
- [ ] Profile can be updated
- [ ] Password can be changed
- [ ] Schedule report shows variance data
- [ ] Budget report shows category breakdown
- [ ] All TypeScript compiles without errors

---

## Agent Execution Order

```
1. Phase 5 must be complete
   |
   +---> 2. Stream A: Dashboard Backend [parallel]
   |
   +---> 3. Stream B: Search & Reports [parallel]
   |
   v
4. Stream C: Frontend (after A & B)
   |
   v
5. Integration & Testing
   |
   v
6. Audit
```

**Parallel execution possible:**
- Streams A and B can run in parallel
- Stream C requires A and B to complete first

# Phase 6: Implementation Checklist

---

## Backend - Dashboard

- [ ] Create `dashboard.controller.ts`
- [ ] Implement aggregation queries:
  - [ ] Active project count
  - [ ] Total budget/spent
  - [ ] Overdue tasks count
  - [ ] Upcoming milestones
  - [ ] Unpaid invoices
- [ ] Get user's projects with metrics
- [ ] Get user's task items (limit 5)
- [ ] Get user's upcoming reminders (limit 5)
- [ ] Get recent activity (limit 10)
- [ ] Get undismissed insights

---

## Backend - Search

- [ ] Create `search.controller.ts`
- [ ] Implement full-text search queries:
  - [ ] Projects: name, address
  - [ ] Tasks: name
  - [ ] Documents: name, extracted_text
  - [ ] Invoices: invoice_number, description
  - [ ] Task items: title
- [ ] Consider PostgreSQL full-text search setup
- [ ] Limit results per type

---

## Backend - Reports

- [ ] Create `reports.controller.ts`
- [ ] Schedule report:
  - [ ] Task variance calculations
  - [ ] Milestone status summary
  - [ ] On-time completion percentage
- [ ] Budget report:
  - [ ] Category breakdown
  - [ ] Phase breakdown
  - [ ] Monthly spending trend
- [ ] Cross-project spending:
  - [ ] By project totals
  - [ ] By category totals
  - [ ] Efficiency comparison

---

## Backend Routes

- [ ] GET `/dashboard`
- [ ] GET `/search`
- [ ] GET `/projects/:id/reports/schedule`
- [ ] GET `/projects/:id/reports/budget`
- [ ] GET `/reports/spending`

---

## Frontend Components

### Dashboard
- [ ] `SummaryCard.tsx`
- [ ] `ProjectCard.tsx`
- [ ] `TaskItemPreview.tsx`
- [ ] `ReminderPreview.tsx`
- [ ] `InsightCard.tsx`
- [ ] `ActivityItem.tsx`
- [ ] `ActivityFeed.tsx`

### Search
- [ ] `SearchModal.tsx`
- [ ] `SearchInput.tsx`
- [ ] `SearchResults.tsx`
- [ ] `SearchResultItem.tsx`
- [ ] Keyboard navigation (↑↓ Enter Esc)

### Settings
- [ ] `ProfileForm.tsx`
- [ ] `PasswordForm.tsx`
- [ ] Settings tab navigation

---

## Frontend Pages

- [ ] Complete `/dashboard/page.tsx`
- [ ] `/settings/page.tsx` (profile)
- [ ] `/settings/security/page.tsx`
- [ ] Verify notifications tab from Phase 5

---

## Frontend Hooks

- [ ] `useDashboard()` - Fetch dashboard data
- [ ] `useSearch(query)` - Search with debounce
- [ ] `useScheduleReport(projectId)`
- [ ] `useBudgetReport(projectId)`
- [ ] `useSpendingReport(filters)`

---

## Search UX

- [ ] Cmd+K / Ctrl+K shortcut to open
- [ ] Debounce input (300ms)
- [ ] Loading state while searching
- [ ] Empty state for no results
- [ ] Keyboard navigation
- [ ] Click or Enter to navigate

---

## Testing

- [ ] Dashboard loads with correct data
- [ ] Summary cards show accurate counts
- [ ] Project cards link to projects
- [ ] Search returns relevant results
- [ ] Search respects builder isolation
- [ ] Reports calculate correctly
- [ ] Settings save correctly
- [ ] Password change works

---

## Definition of Done

- [ ] Dashboard shows multi-project overview
- [ ] All summary cards accurate
- [ ] Project cards show progress
- [ ] Task items and reminders display
- [ ] AI insights show with dismiss
- [ ] Activity feed updates
- [ ] Global search works across types
- [ ] Keyboard shortcuts work
- [ ] Profile can be updated
- [ ] Password can be changed

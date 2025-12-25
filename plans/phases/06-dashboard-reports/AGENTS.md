# Phase 6: Agent Work Instructions

Detailed instructions for parallel agent execution.

---

## Overview

Phase 6 is split into 3 work streams:

| Stream | Description | Dependencies | Estimated Items |
|--------|-------------|--------------|-----------------|
| A | Dashboard Backend | Phase 5 complete | 8 items |
| B | Search & Reports | Phase 5 complete | 12 items |
| C | Frontend Components | Streams A & B | 20 items |

---

## Agent 1: Stream A - Dashboard Backend

**Purpose:** Create the dashboard aggregation endpoint.

**Prompt for Agent:**
```
You are implementing Phase 6 Stream A (Dashboard Backend) for HarvestIQ.

TASK: Create the dashboard controller and routes that aggregate data from all Phase 1-5 tables.

REFERENCE FILES:
- API contracts: /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/06-dashboard-reports/API.md
- Checklist: /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/06-dashboard-reports/CHECKLIST.md (Stream A section)

PATTERN FILES TO FOLLOW:
- Controller pattern: /Users/joey/Projects/X1LLC/HarvestIQ-backend/src/controllers/project.controller.ts

FILES TO CREATE:
Backend (in /Users/joey/Projects/X1LLC/HarvestIQ-backend/):
- src/controllers/dashboard.controller.ts
- src/routes/dashboard.routes.ts

FILES TO MODIFY:
- src/routes/index.ts - Register dashboard routes

AGGREGATION QUERIES NEEDED:
1. Active project count: SELECT COUNT(*) FROM projects WHERE status = 'active' AND builder_id = $1
2. Total budget: SELECT SUM(estimated_amount) FROM budget_items bi JOIN projects p ON bi.project_id = p.id WHERE p.builder_id = $1
3. Total spent: SELECT SUM(actual_amount) FROM budget_items...
4. Overdue tasks: SELECT COUNT(*) FROM task_items WHERE due_date < NOW() AND status != 'completed' AND builder_id = $1
5. Upcoming milestones: SELECT COUNT(*) FROM schedule_milestones WHERE target_date BETWEEN NOW() AND NOW() + interval '30 days'
6. Unpaid invoices: SELECT COUNT(*), SUM(amount - paid_amount) FROM invoices WHERE status != 'paid'

IMPORTANT:
- All queries MUST filter by builder_id for multi-tenant isolation
- All queries MUST filter deleted_at IS NULL
- Return data matching the API.md response format exactly

VERIFICATION:
1. Run: npx tsc --noEmit
2. Start server: npm run dev
3. Test: curl http://localhost:3001/api/dashboard (with auth token)
```

---

## Agent 2: Stream B - Search & Reports Backend

**Purpose:** Create search and report endpoints.

**Prompt for Agent:**
```
You are implementing Phase 6 Stream B (Search & Reports) for HarvestIQ.

TASK: Create search controller and reports controller with their routes.

REFERENCE FILES:
- API contracts: /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/06-dashboard-reports/API.md
- Checklist: /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/06-dashboard-reports/CHECKLIST.md (Stream B section)

FILES TO CREATE:
Backend (in /Users/joey/Projects/X1LLC/HarvestIQ-backend/):
- src/controllers/search.controller.ts
- src/controllers/reports.controller.ts
- src/routes/search.routes.ts
- src/routes/reports.routes.ts

FILES TO MODIFY:
- src/routes/index.ts - Register new routes
- src/routes/project.routes.ts - Add project-specific report routes

SEARCH IMPLEMENTATION:
Use PostgreSQL ILIKE for simple search:
```sql
SELECT id, name, address, status FROM projects
WHERE builder_id = $1 AND deleted_at IS NULL
AND (name ILIKE $2 OR address ILIKE $2)
LIMIT $3
```

REPORT CALCULATIONS:
1. Schedule Report:
   - Variance = actual_end_date - planned_end_date (in days)
   - On-time % = completed_on_time / total_completed * 100

2. Budget Report:
   - By category: GROUP BY category_id
   - Variance = actual - estimated
   - Variance % = (actual - estimated) / estimated * 100

3. Spending Report:
   - By project: SUM(actual_amount) GROUP BY project_id
   - By category: SUM(actual_amount) GROUP BY category_id
   - Cost per unit = total_spent / project.units

IMPORTANT:
- All queries MUST filter by builder_id
- Return data matching the API.md response format

VERIFICATION:
1. Run: npx tsc --noEmit
2. Test search: curl "http://localhost:3001/api/search?q=test"
3. Test reports: curl "http://localhost:3001/api/projects/{id}/reports/budget"
```

---

## Agent 3: Stream C - Frontend Components

**Purpose:** Create all frontend UI components and pages.

**Prerequisites:** Streams A and B must be complete.

**Prompt for Agent:**
```
You are implementing Phase 6 Stream C (Frontend) for HarvestIQ.

TASK: Create frontend hooks, components, and pages for dashboard, search, and settings.

REFERENCE FILES:
- UI wireframes: /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/06-dashboard-reports/UI.md
- API contracts: /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/06-dashboard-reports/API.md
- Checklist: /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/06-dashboard-reports/CHECKLIST.md (Stream C section)

PATTERN FILES TO FOLLOW:
- Hook pattern: /Users/joey/Projects/X1LLC/HarvestIQ/src/hooks/usePayments.ts
- Component pattern: /Users/joey/Projects/X1LLC/HarvestIQ/src/components/payments/

FILES TO CREATE:
Frontend (in /Users/joey/Projects/X1LLC/HarvestIQ/):

Hooks:
- src/hooks/useDashboard.ts
- src/hooks/useSearch.ts
- src/hooks/useScheduleReport.ts
- src/hooks/useBudgetReport.ts
- src/hooks/useSpendingReport.ts

Dashboard Components:
- src/components/dashboard/SummaryCard.tsx
- src/components/dashboard/ProjectCard.tsx
- src/components/dashboard/TaskItemPreview.tsx
- src/components/dashboard/ReminderPreview.tsx
- src/components/dashboard/InsightCard.tsx
- src/components/dashboard/ActivityItem.tsx
- src/components/dashboard/ActivityFeed.tsx

Search Components:
- src/components/search/SearchModal.tsx
- src/components/search/SearchInput.tsx
- src/components/search/SearchResults.tsx
- src/components/search/SearchResultItem.tsx

Settings Components:
- src/components/settings/ProfileForm.tsx
- src/components/settings/PasswordForm.tsx
- src/components/settings/SettingsTabs.tsx

Pages:
- src/app/settings/page.tsx
- src/app/settings/security/page.tsx

FILES TO MODIFY:
- src/app/dashboard/page.tsx - Replace with full dashboard
- src/components/DashboardLayout.tsx - Add search trigger and Cmd+K listener

KEYBOARD SHORTCUT IMPLEMENTATION:
```typescript
useEffect(() => {
  const handleKeyDown = (e: KeyboardEvent) => {
    if ((e.metaKey || e.ctrlKey) && e.key === 'k') {
      e.preventDefault();
      setSearchOpen(true);
    }
  };
  document.addEventListener('keydown', handleKeyDown);
  return () => document.removeEventListener('keydown', handleKeyDown);
}, []);
```

VERIFICATION:
1. Run: npx tsc --noEmit
2. Start dev server: npm run dev
3. Navigate to /dashboard
4. Press Cmd+K to open search
5. Navigate to /settings
```

---

## Agent 4: Integration Testing

**Purpose:** Verify all features work end-to-end.

**Prerequisites:** Streams A, B, and C must all be complete.

**Prompt for Agent:**
```
You are performing integration testing for Phase 6 of HarvestIQ.

TASK: Test all Phase 6 features and verify they work correctly.

CHECKLIST:
Read /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/06-dashboard-reports/CHECKLIST.md
(see "Integration & Testing" section)

TESTS TO PERFORM:
1. Start both servers:
   - Backend: cd /Users/joey/Projects/X1LLC/HarvestIQ-backend && npm run dev
   - Frontend: cd /Users/joey/Projects/X1LLC/HarvestIQ && npm run dev

2. Test Dashboard:
   - Navigate to /dashboard
   - Verify summary cards show correct counts
   - Verify project cards display
   - Verify task items and reminders show
   - Verify activity feed loads

3. Test Search:
   - Press Cmd+K (Mac) or Ctrl+K (Windows)
   - Type a search query
   - Verify results appear
   - Use keyboard navigation (↑↓)
   - Press Enter to navigate to result
   - Press Esc to close

4. Test Settings:
   - Navigate to /settings
   - Update profile name
   - Verify it saves
   - Navigate to /settings/security
   - Test password change

5. Test Reports:
   - Navigate to a project
   - View schedule report
   - View budget report
   - Verify calculations are correct

REPORT:
Create a report of all tests performed and their results.
Note any bugs or issues found.
```

---

## Agent 5: Audit

**Purpose:** Final verification that Phase 6 is complete.

**Prerequisites:** Integration testing must be complete.

**Prompt for Agent:**
```
You are auditing Phase 6 implementation for HarvestIQ.

TASK: Verify all Phase 6 deliverables are complete and working.

CHECKLIST:
Read /Users/joey/Projects/X1LLC/HarvestIQ-docs/plans/phases/06-dashboard-reports/CHECKLIST.md
(see "Audit Checklist" and "Definition of Done" sections)

VERIFICATION STEPS:
1. Check TypeScript compilation:
   - cd /Users/joey/Projects/X1LLC/HarvestIQ-backend && npx tsc --noEmit
   - cd /Users/joey/Projects/X1LLC/HarvestIQ && npx tsc --noEmit

2. Check API endpoints:
   - GET /dashboard - Returns aggregated data
   - GET /search?q=test - Returns search results
   - GET /projects/{id}/reports/schedule - Returns schedule report
   - GET /projects/{id}/reports/budget - Returns budget report
   - GET /reports/spending - Returns spending analysis

3. Check frontend pages:
   - /dashboard renders with data
   - /settings renders profile form
   - /settings/security renders password form
   - Cmd+K opens search modal

4. Verify data accuracy:
   - Compare dashboard counts to database queries
   - Verify report calculations manually

REPORT:
Create a Phase 6 audit report with:
- All items verified
- Any issues found
- Recommendations for fixes
```

---

## Execution Summary

| Order | Agent | Stream | Can Start After |
|-------|-------|--------|-----------------|
| 1 | Stream A | Dashboard Backend | Phase 5 complete |
| 1 | Stream B | Search & Reports | Phase 5 complete |
| 2 | Stream C | Frontend | Streams A & B |
| 3 | Integration | Testing | All streams |
| 4 | Audit | Verification | Integration |

**Note:** Agents 1 (Stream A) and 2 (Stream B) can run in parallel.

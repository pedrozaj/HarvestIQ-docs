# Phase 3: Implementation Checklist

---

## Database

- [ ] Create migration: `010_create_budget_categories.sql`
- [ ] Create migration: `011_create_schedule_phases.sql`
- [ ] Create migration: `012_create_schedule_milestones.sql`
- [ ] Create migration: `013_create_schedule_tasks.sql`
- [ ] Create migration: `014_create_task_dependencies.sql`
- [ ] Create migration: `015_create_budget_items.sql`
- [ ] Add default budget categories seeding to builder creation
- [ ] Run migrations and verify

> **Note:** Migrations 001-009 were created in Phases 1-2.

---

## Backend Models

- [ ] `budgetCategory.model.ts` - CRUD + findByBuilder
- [ ] `schedulePhase.model.ts` - CRUD + reorder
- [ ] `scheduleTask.model.ts` - CRUD + filters + dependencies
- [ ] `scheduleMilestone.model.ts` - CRUD
- [ ] `taskDependency.model.ts` - add/remove + circular check
- [ ] `budgetItem.model.ts` - CRUD + summary

---

## Backend Controllers

- [ ] `budgetCategory.controller.ts`
- [ ] `schedulePhase.controller.ts`
- [ ] `scheduleTask.controller.ts`
- [ ] `scheduleMilestone.controller.ts`
- [ ] `budgetItem.controller.ts`

---

## Backend Routes

- [ ] `/budget-categories` routes
- [ ] `/projects/:id/phases` routes
- [ ] `/projects/:id/tasks` routes
- [ ] `/projects/:id/milestones` routes
- [ ] `/projects/:id/timeline` route
- [ ] `/projects/:id/budget` routes

---

## Validation Schemas

- [ ] `budgetCategorySchema`
- [ ] `schedulePhaseSchema`
- [ ] `scheduleTaskSchema`
- [ ] `scheduleMilestoneSchema`
- [ ] `budgetItemSchema`
- [ ] `dependencySchema`

---

## Circular Dependency Check

- [ ] Implement graph traversal function
- [ ] Validate before adding dependency
- [ ] Return clear error message

---

## Frontend Components

### Schedule
- [ ] `TaskList.tsx` - Grouped by phase
- [ ] `TaskRow.tsx` - Task display with status
- [ ] `TaskForm.tsx` - Create/edit modal
- [ ] `PhaseHeader.tsx` - Collapsible header
- [ ] `DependencySelect.tsx` - Multi-select

### Gantt
- [ ] Install gantt library (frappe-gantt)
- [ ] `GanttChart.tsx` - Timeline component
- [ ] Integrate with task data

### Calendar
- [ ] Install react-big-calendar
- [ ] `CalendarView.tsx` - Calendar component
- [ ] Task and milestone events

### Milestones
- [ ] `MilestoneList.tsx` - Grouped display
- [ ] `MilestoneForm.tsx` - Create/edit modal
- [ ] `MilestoneCard.tsx` - Individual milestone

### Budget
- [ ] `BudgetOverview.tsx` - Summary + chart
- [ ] `BudgetTable.tsx` - Items list
- [ ] `BudgetItemForm.tsx` - Create/edit modal
- [ ] `CategorySelect.tsx` - Category dropdown
- [ ] Install recharts for pie chart

---

## Frontend Pages

- [ ] Update project detail to add Schedule tab
- [ ] Create schedule sub-routes structure
- [ ] `/projects/[id]/schedule/page.tsx` (task list)
- [ ] `/projects/[id]/schedule/gantt/page.tsx`
- [ ] `/projects/[id]/schedule/calendar/page.tsx`
- [ ] `/projects/[id]/schedule/milestones/page.tsx`
- [ ] `/projects/[id]/budget/page.tsx`

---

## Frontend Hooks

- [ ] `useBudgetCategories()` - List categories
- [ ] `usePhases(projectId)` - List phases
- [ ] `useTasks(projectId, filters)` - List tasks
- [ ] `useMilestones(projectId)` - List milestones
- [ ] `useTimeline(projectId)` - Gantt data
- [ ] `useBudgetItems(projectId, filters)` - Budget items
- [ ] `useBudgetSummary(projectId)` - Summary data
- [ ] Mutation hooks for create/update/delete

---

## Constants Updates

Add to `src/constants/index.ts`:

```typescript
ENTITY_TYPE: {
  // existing...
  PHASE: 'phase',
  TASK: 'task',
  MILESTONE: 'milestone',
  BUDGET_ITEM: 'budget_item',
  BUDGET_CATEGORY: 'budget_category',
}

ACTIVITY_ACTION: {
  // existing...
  COMPLETED: 'completed',
  ACHIEVED: 'achieved',
  DEPENDENCY_ADDED: 'dependency_added',
  DEPENDENCY_REMOVED: 'dependency_removed',
}
```

---

## Activity Logging

- [ ] Log phase created/updated/deleted
- [ ] Log task created/updated/completed/deleted
- [ ] Log milestone created/achieved/deleted
- [ ] Log budget item created/updated/deleted
- [ ] Log dependency added/removed

---

## Testing

- [ ] Test phase CRUD and reorder
- [ ] Test task CRUD with dependencies
- [ ] Test circular dependency prevention
- [ ] Test milestone status updates
- [ ] Test budget calculations
- [ ] Test Gantt chart rendering
- [ ] Test calendar view

---

## Definition of Done

- [ ] Categories can be managed at builder level
- [ ] Phases can be created, edited, reordered
- [ ] Tasks can be created with all fields
- [ ] Dependencies work and prevent circular refs
- [ ] Task status can be updated inline
- [ ] Milestones track target vs actual
- [ ] Budget items calculate variance
- [ ] Budget summary shows totals correctly
- [ ] Gantt chart displays all data
- [ ] Calendar shows tasks and milestones
- [ ] All actions logged to activity

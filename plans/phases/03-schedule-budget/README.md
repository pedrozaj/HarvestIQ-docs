# Phase 3: Schedule & Budget

Core construction management features for tracking work and costs.

---

## Phase Goal

Enable full schedule management with phases, tasks, milestones, and dependencies. Enable budget tracking with categories and line items.

---

## Dependencies

- Phase 1: Infrastructure (complete)
- Phase 2: Core Data Models (complete) - migrations 001-009

---

## Deliverables

1. Budget categories (builder-level)
2. Schedule phases per project
3. Tasks with dependencies and assignments
4. Milestones with tracking
5. Budget line items with estimated/actual
6. Gantt chart visualization
7. Calendar view

---

## Tables Introduced

| Table | Purpose |
|-------|---------|
| `budget_categories` | Cost categories (plumbing, electrical, etc.) |
| `schedule_phases` | Project phases (Foundation, Framing, etc.) |
| `schedule_milestones` | Key date targets |
| `schedule_tasks` | Individual work items |
| `task_dependencies` | Task dependency links |
| `budget_items` | Individual budget line items |

---

## Endpoints Introduced

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET/POST | `/budget-categories` | Manage categories |
| GET/POST | `/projects/:id/phases` | Manage phases |
| PUT | `/projects/:id/phases/reorder` | Reorder phases |
| GET/POST/PUT/DELETE | `/projects/:id/tasks` | Task CRUD |
| POST/DELETE | `/projects/:id/tasks/:id/dependencies` | Dependencies |
| GET/POST/PUT/DELETE | `/projects/:id/milestones` | Milestone CRUD |
| GET | `/projects/:id/timeline` | Gantt data |
| GET/POST/PUT/DELETE | `/projects/:id/budget` | Budget items |
| GET | `/projects/:id/budget/summary` | Budget totals |

---

## UI Pages Introduced

| Route | Page | Purpose |
|-------|------|---------|
| /projects/:id/schedule | Schedule Tab | Task management |
| /projects/:id/schedule/gantt | Gantt View | Timeline visualization |
| /projects/:id/schedule/calendar | Calendar View | Date-based view |
| /projects/:id/schedule/milestones | Milestones | Milestone list |
| /projects/:id/budget | Budget Tab | Budget management |

---

## Acceptance Criteria

- [ ] Categories can be created at builder level
- [ ] Phases can be created and reordered
- [ ] Tasks can be created with assignments
- [ ] Task dependencies prevent circular references
- [ ] Milestones track target vs actual dates
- [ ] Budget items link to categories and phases
- [ ] Gantt chart displays correctly
- [ ] Calendar shows tasks and milestones
- [ ] Budget summary calculates variance

---

## Related Documentation

- Schema details: [SCHEMA.md](./SCHEMA.md)
- API endpoints: [API.md](./API.md)
- UI wireframes: [UI.md](./UI.md)
- Implementation checklist: [CHECKLIST.md](./CHECKLIST.md)

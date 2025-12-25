# Phase 6: Dashboard & Reports

Aggregated views, global search, and user settings.

---

## Phase Goal

Build the dashboard with multi-project overview, implement global search, and complete user settings pages.

---

## Prerequisites

| Requirement | Status |
|-------------|--------|
| Phase 1-4: Core Infrastructure | Complete |
| Phase 5: Tasks & Notifications | Required |
| Migrations 001-025 | Required |
| Task items data available | Required |
| Reminders data available | Required |

---

## Deliverables

1. Dashboard with summary cards and project overview
2. Global search across all entities
3. Settings pages (profile, security, notifications)
4. Schedule and budget reports
5. Cross-project spending analysis

---

## Tables Introduced

None - uses existing tables with aggregation queries.

---

## Work Streams

Phase 6 is organized into parallel work streams:

| Stream | Description | Dependencies |
|--------|-------------|--------------|
| **A** | Dashboard Backend & Queries | Phase 5 complete |
| **B** | Search & Reports Backend | Phase 5 complete |
| **C** | Frontend Components | Streams A & B |

```
Phase 5 Complete
     |
     +---> Stream A (Dashboard Backend) [parallel]
     |
     +---> Stream B (Search & Reports) [parallel]
     |
     v
Stream C (Frontend)
     |
     v
Integration & Testing
     |
     v
Audit
```

See [AGENTS.md](./AGENTS.md) for detailed agent work instructions.

---

## Endpoints Introduced

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/dashboard` | Aggregated dashboard data |
| GET | `/search` | Global search |
| GET | `/projects/:id/reports/schedule` | Schedule report |
| GET | `/projects/:id/reports/budget` | Budget report |
| GET | `/reports/spending` | Cross-project spending |

---

## UI Pages Introduced

| Route | Page | Purpose |
|-------|------|---------|
| /dashboard | Dashboard | Multi-project overview |
| /settings | Settings | User profile |
| /settings/security | Security | Password change |

---

## Acceptance Criteria

- [ ] Dashboard shows accurate summary stats
- [ ] Project cards display key metrics
- [ ] Global search finds across entity types
- [ ] Keyboard shortcut (Cmd+K) opens search
- [ ] Profile can be updated
- [ ] Password can be changed
- [ ] Reports show correct data

---

## Related Documentation

| Document | Purpose |
|----------|---------|
| [API.md](./API.md) | API endpoint contracts |
| [UI.md](./UI.md) | UI wireframes |
| [CHECKLIST.md](./CHECKLIST.md) | Implementation checklist |
| [AGENTS.md](./AGENTS.md) | Agent work instructions |

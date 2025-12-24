# Phase 6: Dashboard & Reports

Aggregated views, global search, and user settings.

---

## Phase Goal

Build the dashboard with multi-project overview, implement global search, and complete user settings pages.

---

## Dependencies

- Phases 3, 4, 5 complete (needs data from all)

---

## Deliverables

1. Dashboard with summary cards and project overview
2. Global search across all entities
3. Settings pages (profile, security, notifications)
4. Schedule and budget reports

---

## Tables Introduced

None - uses existing tables with aggregation queries.

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
- [ ] Profile can be updated
- [ ] Password can be changed
- [ ] Reports show correct data

---

## Related Documentation

- [API.md](./API.md) | [UI.md](./UI.md) | [CHECKLIST.md](./CHECKLIST.md)

# Phase 4: Documents & Payments

File storage, invoice tracking, and payment management.

---

## Phase Goal

Enable document uploads to R2 storage, invoice tracking with payment status, and payment recording linked to budgets.

---

## Dependencies

- Phase 3: Schedule & Budget (complete) - needs budget categories

---

## Deliverables

1. Document upload with R2 signed URLs
2. Document preview and download
3. Invoice management with status tracking
4. Payment recording linked to invoices/categories
5. Aging report for accounts receivable

---

## Tables Introduced

| Table | Purpose |
|-------|---------|
| `documents` | File metadata and storage references |
| `invoices` | Invoice records with payment status |
| `payments` | Payment transaction records |

---

## Endpoints Introduced

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET/POST | `/projects/:id/documents` | Document list/upload |
| POST | `/projects/:id/documents/upload-url` | Get signed upload URL |
| GET | `/projects/:id/documents/:id/download` | Get signed download URL |
| PUT | `/projects/:id/documents/:id/tags` | Manage tags |
| GET/POST | `/projects/:id/invoices` | Invoice management |
| GET | `/invoices/aging` | Aging report |
| GET/POST | `/projects/:id/payments` | Payment management |
| GET | `/payments/outstanding` | Outstanding balances |

---

## UI Pages Introduced

| Route | Page | Purpose |
|-------|------|---------|
| /projects/:id/documents | Documents Tab | File management |
| /projects/:id/payments | Payments Tab | Invoices & payments |

---

## Acceptance Criteria

- [ ] Files upload to R2 with signed URLs
- [ ] Documents display with preview
- [ ] Documents can be tagged
- [ ] Invoices track payment status
- [ ] Payments link to invoices/categories
- [ ] Aging report shows overdue invoices
- [ ] Outstanding balances calculate correctly

---

## Related Documentation

- [SCHEMA.md](./SCHEMA.md) | [API.md](./API.md) | [UI.md](./UI.md) | [CHECKLIST.md](./CHECKLIST.md)

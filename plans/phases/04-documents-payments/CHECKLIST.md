# Phase 4: Implementation Checklist

---

## Database

- [ ] Create migration: `013_create_documents.sql`
- [ ] Create migration: `014_create_invoices.sql`
- [ ] Create migration: `015_create_payments.sql`
- [ ] Run migrations and verify

---

## R2 Storage Setup

- [ ] Configure R2 bucket credentials
- [ ] Create `src/services/storage.service.ts`
- [ ] Implement `getSignedUploadUrl(key, mimeType)`
- [ ] Implement `getSignedDownloadUrl(key)`
- [ ] Implement `deleteObject(key)`
- [ ] Test upload/download flow

---

## Backend Models

- [ ] `document.model.ts` - CRUD + tag management
- [ ] `invoice.model.ts` - CRUD + status calc + aging
- [ ] `payment.model.ts` - CRUD + invoice status update

---

## Backend Controllers

- [ ] `document.controller.ts`
  - [ ] getUploadUrl
  - [ ] create (after upload)
  - [ ] list with filters
  - [ ] getDownloadUrl
  - [ ] updateTags
  - [ ] delete
- [ ] `invoice.controller.ts`
  - [ ] CRUD operations
  - [ ] aging report
- [ ] `payment.controller.ts`
  - [ ] CRUD operations
  - [ ] outstanding balances

---

## Backend Routes

- [ ] `/projects/:id/documents` routes
- [ ] `/projects/:id/invoices` routes
- [ ] `/projects/:id/payments` routes
- [ ] `/invoices/aging` route
- [ ] `/payments/outstanding` route

---

## Validation Schemas

- [ ] `documentUploadSchema` (file validation)
- [ ] `documentCreateSchema`
- [ ] `invoiceSchema`
- [ ] `paymentSchema`

---

## Frontend Components

### Documents
- [ ] `DocumentGrid.tsx`
- [ ] `DocumentCard.tsx`
- [ ] `DocumentUpload.tsx` (drag-drop)
- [ ] `DocumentPreview.tsx` (modal)
- [ ] `TagInput.tsx`

### Invoices
- [ ] `InvoiceTable.tsx`
- [ ] `InvoiceForm.tsx`
- [ ] `InvoiceDetail.tsx`
- [ ] `AgingReport.tsx`

### Payments
- [ ] `PaymentForm.tsx`
- [ ] `PaymentTable.tsx`

---

## Frontend Pages

- [ ] `/projects/[id]/documents/page.tsx`
- [ ] `/projects/[id]/payments/page.tsx`

---

## Frontend Hooks

- [ ] `useDocuments(projectId, filters)`
- [ ] `useUploadDocument()` - upload flow
- [ ] `useInvoices(projectId, filters)`
- [ ] `usePayments(projectId, filters)`
- [ ] `useAgingReport()`
- [ ] `useOutstandingBalances()`

---

## File Upload Flow

1. [ ] Frontend requests signed upload URL
2. [ ] Frontend uploads directly to R2
3. [ ] Frontend calls POST /documents with metadata
4. [ ] Backend creates document record
5. [ ] Document appears in list

---

## Invoice → Payment Flow

1. [ ] Create invoice with amount
2. [ ] Record payment (partial or full)
3. [ ] Auto-calculate paid_amount
4. [ ] Auto-update invoice status
5. [ ] Display in payment history

---

## Activity Logging

- [ ] Log document uploaded/deleted
- [ ] Log invoice created/updated/deleted
- [ ] Log payment recorded/deleted

---

## Testing

- [ ] Test file upload to R2
- [ ] Test file download with signed URL
- [ ] Test document filtering by type/tags
- [ ] Test invoice CRUD
- [ ] Test payment recording
- [ ] Test invoice status auto-update
- [ ] Test aging report calculations
- [ ] Test outstanding balances

---

## Definition of Done

- [ ] Files upload to R2 successfully
- [ ] Documents display in grid/list
- [ ] PDF/image preview works
- [ ] Tags can be added/removed
- [ ] Invoices show payment status
- [ ] Payments update invoice status
- [ ] Aging report shows correct buckets
- [ ] Outstanding balances calculate correctly
- [ ] All actions logged to activity

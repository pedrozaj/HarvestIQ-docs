# Phase 4: Database Schema

Document, invoice, and payment tables.

---

## Migration Order

```
017_create_documents.sql
018_create_invoices.sql
019_create_payments.sql
```

> **Note**: Migrations 013-016 are used by Phase 3 (Schedule & Budget).

---

## Table: documents

File metadata with R2 storage references.

```sql
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  uploaded_by UUID NOT NULL REFERENCES users(id),
  name VARCHAR(255) NOT NULL,
  original_filename VARCHAR(255) NOT NULL,
  storage_key VARCHAR(500) NOT NULL,
    -- R2 object key: {builder_id}/{project_id}/{uuid}/{filename}
  mime_type VARCHAR(100) NOT NULL,
  size_bytes BIGINT NOT NULL,
  type VARCHAR(50) NOT NULL DEFAULT 'other',
    -- values: invoice, estimate, contract, permit, photo, other
  description TEXT,
  tags TEXT[] DEFAULT '{}',
  extracted_text TEXT,
    -- For AI search (Phase 7)
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_documents_project_id ON documents(project_id);
CREATE INDEX idx_documents_type ON documents(type);
CREATE INDEX idx_documents_tags ON documents USING GIN(tags);
CREATE INDEX idx_documents_deleted_at ON documents(deleted_at);
```

---

## Table: invoices

Invoice records with payment tracking.

```sql
CREATE TABLE invoices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  category_id UUID REFERENCES budget_categories(id),
  document_id UUID REFERENCES documents(id) ON DELETE SET NULL,
  invoice_number VARCHAR(100),
  description TEXT,
  amount DECIMAL(12,2) NOT NULL,
  paid_amount DECIMAL(12,2) NOT NULL DEFAULT 0,
  status VARCHAR(50) NOT NULL DEFAULT 'unpaid',
    -- values: unpaid, partial, paid
  invoice_date DATE,
  due_date DATE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_invoices_project_id ON invoices(project_id);
CREATE INDEX idx_invoices_category_id ON invoices(category_id);
CREATE INDEX idx_invoices_status ON invoices(status);
CREATE INDEX idx_invoices_due_date ON invoices(due_date);
```

---

## Table: payments

Payment transaction records.

```sql
CREATE TABLE payments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  invoice_id UUID REFERENCES invoices(id) ON DELETE SET NULL,
  category_id UUID REFERENCES budget_categories(id),
  amount DECIMAL(12,2) NOT NULL,
  payment_date DATE NOT NULL,
  payment_method VARCHAR(50),
    -- values: check, ach, credit_card, wire, cash, other
  reference_number VARCHAR(100),
  description TEXT,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_payments_project_id ON payments(project_id);
CREATE INDEX idx_payments_invoice_id ON payments(invoice_id);
CREATE INDEX idx_payments_category_id ON payments(category_id);
CREATE INDEX idx_payments_payment_date ON payments(payment_date);
```

---

## Key Queries

### Aging Report

```sql
SELECT
  CASE
    WHEN due_date >= CURRENT_DATE THEN 'current'
    WHEN due_date >= CURRENT_DATE - INTERVAL '30 days' THEN '1-30'
    WHEN due_date >= CURRENT_DATE - INTERVAL '60 days' THEN '31-60'
    WHEN due_date >= CURRENT_DATE - INTERVAL '90 days' THEN '61-90'
    ELSE '90+'
  END as aging_bucket,
  COUNT(*) as invoice_count,
  SUM(amount - paid_amount) as outstanding
FROM invoices
WHERE builder_id = $1
AND status != 'paid'
AND deleted_at IS NULL
GROUP BY aging_bucket;
```

### Outstanding by Project

```sql
SELECT
  p.id, p.name,
  COUNT(i.id) as invoice_count,
  SUM(i.amount - i.paid_amount) as outstanding
FROM projects p
LEFT JOIN invoices i ON i.project_id = p.id AND i.status != 'paid'
WHERE p.builder_id = $1 AND p.deleted_at IS NULL
GROUP BY p.id
HAVING SUM(i.amount - i.paid_amount) > 0;
```

---

## Payment → Invoice Status Update

When payment is recorded against invoice, recalculate status:

```sql
UPDATE invoices
SET
  paid_amount = (SELECT COALESCE(SUM(amount), 0) FROM payments WHERE invoice_id = $1),
  status = CASE
    WHEN paid_amount >= amount THEN 'paid'
    WHEN paid_amount > 0 THEN 'partial'
    ELSE 'unpaid'
  END,
  updated_at = NOW()
WHERE id = $1;
```

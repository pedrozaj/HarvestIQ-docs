# Phase 4: API Endpoints

Document and payment management endpoints.

---

## Documents

### POST /projects/:id/documents/upload-url

Get signed URL for direct upload to R2.

```typescript
Request: {
  filename: string;
  mimeType: string;
  size: number;
}

Response: {
  data: {
    uploadUrl: string;      // Signed PUT URL
    storageKey: string;     // Key to use in POST /documents
    expiresIn: number;      // Seconds until expiry
  }
}
```

### POST /projects/:id/documents

Create document record after upload complete.

```typescript
Request: {
  storageKey: string;       // From upload-url response
  name: string;
  originalFilename: string;
  mimeType: string;
  sizeBytes: number;
  type: "invoice" | "estimate" | "contract" | "permit" | "photo" | "other";
  description?: string;
  tags?: string[];
}

Response: { data: { id, name, type, createdAt, ... } }
```

### GET /projects/:id/documents

List documents with filters.

```typescript
Query: {
  type?: string;
  tags?: string;          // Comma-separated
  search?: string;
  limit?: number;
  offset?: number;
}

Response: {
  data: [{
    id: string;
    name: string;
    type: string;
    mimeType: string;
    sizeBytes: number;
    tags: string[];
    uploadedBy: { id, name };
    createdAt: string;
  }],
  total: number
}
```

### GET /projects/:id/documents/:docId

Get document details.

### GET /projects/:id/documents/:docId/download

Get signed download URL.

```typescript
Response: {
  data: {
    downloadUrl: string;
    expiresIn: number;
  }
}
```

### PUT /projects/:id/documents/:docId

Update document metadata.

### PUT /projects/:id/documents/:docId/tags

Update document tags.

```typescript
Request: { tags: string[] }
```

### DELETE /projects/:id/documents/:docId

Soft delete document.

---

## Invoices

### GET /projects/:id/invoices

List invoices with filters.

```typescript
Query: {
  status?: "unpaid" | "partial" | "paid";
  categoryId?: string;
  startDate?: string;
  endDate?: string;
  limit?: number;
  offset?: number;
}

Response: {
  data: [{
    id: string;
    invoiceNumber: string | null;
    description: string | null;
    category: { id, name } | null;
    document: { id, name } | null;
    amount: number;
    paidAmount: number;
    status: string;
    invoiceDate: string | null;
    dueDate: string | null;
    isOverdue: boolean;
    daysOverdue: number | null;
  }],
  total: number
}
```

### POST /projects/:id/invoices

Create invoice.

```typescript
Request: {
  invoiceNumber?: string;
  description?: string;
  categoryId?: string;
  documentId?: string;
  amount: number;
  invoiceDate?: string;
  dueDate?: string;
}
```

### GET /projects/:id/invoices/:invId

Get invoice with payment history.

```typescript
Response: {
  data: {
    ...invoice,
    payments: [{
      id: string;
      amount: number;
      paymentDate: string;
      paymentMethod: string;
      referenceNumber: string | null;
    }]
  }
}
```

### PUT /projects/:id/invoices/:invId

Update invoice.

### DELETE /projects/:id/invoices/:invId

Delete invoice.

### GET /invoices/aging

Get aging report across all projects.

```typescript
Response: {
  data: {
    current: { count: number; amount: number };
    days1to30: { count: number; amount: number };
    days31to60: { count: number; amount: number };
    days61to90: { count: number; amount: number };
    days90plus: { count: number; amount: number };
    total: { count: number; amount: number };
    byProject: [{
      projectId: string;
      projectName: string;
      outstanding: number;
    }];
    byCategory: [{
      categoryId: string;
      categoryName: string;
      outstanding: number;
    }];
  }
}
```

---

## Payments

### GET /projects/:id/payments

List payments.

```typescript
Query: {
  categoryId?: string;
  startDate?: string;
  endDate?: string;
  paymentMethod?: string;
  limit?: number;
  offset?: number;
}

Response: {
  data: [{
    id: string;
    amount: number;
    paymentDate: string;
    paymentMethod: string;
    referenceNumber: string | null;
    description: string | null;
    category: { id, name } | null;
    invoice: { id, invoiceNumber } | null;
  }],
  total: number
}
```

### POST /projects/:id/payments

Record payment.

```typescript
Request: {
  amount: number;
  paymentDate: string;
  paymentMethod?: string;
  referenceNumber?: string;
  description?: string;
  invoiceId?: string;
  categoryId?: string;
}
```

**Side Effect:** If invoiceId provided, recalculate invoice paid_amount and status.

### PUT /projects/:id/payments/:payId

Update payment.

### DELETE /projects/:id/payments/:payId

Delete payment.

### GET /payments/outstanding

Get outstanding balances summary.

```typescript
Response: {
  data: {
    totalOutstanding: number;
    byProject: [{
      projectId: string;
      projectName: string;
      invoiceCount: number;
      outstanding: number;
    }];
  }
}
```

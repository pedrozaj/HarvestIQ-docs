# Phase 4: UI Pages

Document and payment interfaces.

---

## Documents Tab

```
+-----------------------------------------------------------------------------------+
|  [Overview] [Schedule] [Budget] [Documents] [Payments] [Tasks]                    |
|  ================================================================                 |
|                                                                                   |
|  DOCUMENTS                                    [Grid] [List]      [+ Upload]       |
|                                                                                   |
|  Type: [All v]  Tags: [___________]  [________Search______]                       |
|                                                                                   |
|  +-----------------------------------------------------------------------+        |
|  | +-------------+  +-------------+  +-------------+  +-------------+   |        |
|  | | [PDF icon]  |  | [IMG icon]  |  | [PDF icon]  |  | [PDF icon]  |   |        |
|  | | Plumbing    |  | Site Photo  |  | Contract    |  | Permit      |   |        |
|  | | Invoice.pdf |  | 001.jpg     |  | HVAC.pdf    |  | Building.pdf|   |        |
|  | | Invoice     |  | Photo       |  | Contract    |  | Permit      |   |        |
|  | | Dec 15      |  | Dec 14      |  | Dec 10      |  | Dec 1       |   |        |
|  | +-------------+  +-------------+  +-------------+  +-------------+   |        |
|  +-----------------------------------------------------------------------+        |
|                                                                                   |
|  Showing 1-12 of 24                                 [< Prev] [1] [2] [Next >]     |
+-----------------------------------------------------------------------------------+
```

### Upload Zone

Drag-and-drop area with progress:

```
+------------------------------------------------------------------+
|                                                                   |
|  +-----------------------------------------------------------+   |
|  |                                                           |   |
|  |       [Upload Icon]                                       |   |
|  |                                                           |   |
|  |       Drag files here or click to browse                  |   |
|  |       PDF, Images up to 10MB                              |   |
|  |                                                           |   |
|  +-----------------------------------------------------------+   |
|                                                                   |
|  Uploading...                                                     |
|  Invoice_Dec.pdf          [==============-----] 75%              |
|  SitePhoto.jpg            [===================] Done ✓           |
|                                                                   |
+------------------------------------------------------------------+
```

### Document Preview Modal

```
+------------------------------------------------------------------+
|  Plumbing Invoice.pdf                                       [X]   |
+------------------------------------------------------------------+
|                                                                   |
|  +-----------------------------------------------------------+   |
|  |                                                           |   |
|  |                   [PDF Preview]                           |   |
|  |                                                           |   |
|  +-----------------------------------------------------------+   |
|                                                                   |
|  Type: Invoice           Uploaded by: Joey Smith                  |
|  Size: 1.2 MB            Uploaded: Dec 15, 2024                   |
|                                                                   |
|  Tags: [plumbing] [invoice] [+ Add tag]                           |
|                                                                   |
|  Related Invoice: INV-001 (linked)                                |
|                                                                   |
|                              [Download]  [Delete]                 |
+------------------------------------------------------------------+
```

---

## Payments Tab

```
+-----------------------------------------------------------------------------------+
|  [Overview] [Schedule] [Budget] [Documents] [Payments] [Tasks]                    |
|  ================================================================                 |
|                                                                                   |
|  [Invoices]  [Payments]  [Aging Report]                     [+ Create Invoice]    |
|  ======================================                                           |
|                                                                                   |
|  INVOICES                                                                         |
|  Status: [All v]  Category: [All v]  [________Search______]                       |
|                                                                                   |
|  +-----------------------------------------------------------------------+        |
|  | Invoice #    | Category    | Amount    | Paid      | Status  | Due   |        |
|  +-----------------------------------------------------------------------+        |
|  | INV-001      | Plumbing    | $15,000   | $0        | [Unpaid]| Dec 20|        |
|  |              | Overdue 3 days                                        |        |
|  +-----------------------------------------------------------------------+        |
|  | INV-002      | Electrical  | $8,500    | $5,000    | [Partial]| Dec 25|       |
|  +-----------------------------------------------------------------------+        |
|  | INV-003      | Framing     | $22,000   | $22,000   | [Paid]  | Dec 10|        |
|  +-----------------------------------------------------------------------+        |
+-----------------------------------------------------------------------------------+
```

### Invoice Detail

```
+------------------------------------------------------------------+
|  Invoice INV-001                                            [X]   |
+------------------------------------------------------------------+
|                                                                   |
|  Category: Plumbing                                               |
|  Amount: $15,000.00                                               |
|  Status: [Unpaid] (3 days overdue)                                |
|                                                                   |
|  Invoice Date: Dec 15, 2024                                       |
|  Due Date: Dec 20, 2024                                           |
|                                                                   |
|  Description: Rough-in plumbing for all units                     |
|                                                                   |
|  Linked Document: [Plumbing Invoice.pdf]                          |
|                                                                   |
|  PAYMENT HISTORY                                                  |
|  +-----------------------------------------------------------+   |
|  | No payments recorded                                       |   |
|  +-----------------------------------------------------------+   |
|                                                                   |
|              [Record Payment]  [Edit]  [Delete]                   |
+------------------------------------------------------------------+
```

### Record Payment Modal

```
+------------------------------------------------------------------+
|  Record Payment                                             [X]   |
+------------------------------------------------------------------+
|                                                                   |
|  Invoice: INV-001 - Plumbing ($15,000)                            |
|  Outstanding: $15,000.00                                          |
|                                                                   |
|  Amount *                                                         |
|  [$  15,000.00        ]          [Pay Full Amount]                |
|                                                                   |
|  Payment Date *                                                   |
|  [12/23/2024          ]                                           |
|                                                                   |
|  Payment Method                                                   |
|  [Check                                             v]            |
|                                                                   |
|  Reference # (Check #, Transaction ID)                            |
|  [1234                                               ]            |
|                                                                   |
|  Notes                                                            |
|  [_______________________________________________________]        |
|                                                                   |
|                                [Cancel]  [Record Payment]         |
+------------------------------------------------------------------+
```

---

## Components to Build

| Component | Purpose |
|-----------|---------|
| DocumentGrid | Grid view of documents |
| DocumentCard | Individual document tile |
| DocumentUpload | Drag-drop upload zone |
| DocumentPreview | Preview modal with PDF/image |
| InvoiceTable | Invoice list with status |
| InvoiceForm | Create/edit invoice |
| InvoiceDetail | Invoice with payment history |
| PaymentForm | Record payment modal |
| PaymentTable | Payment history list |
| AgingReport | Aging breakdown display |

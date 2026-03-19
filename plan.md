# Fix Plan: Email Bugs + Document Type Change

## Bug 1: Budget Double-Counting

**Root cause**: `budgetItem.controller.ts:formatBudgetItem()` adds `actual_amount + invoice_paid`, but `actual_amount` is manually set and `invoice_paid` is the SUM of linked invoices — when both exist they represent the same money.

**Fix**: In `formatBudgetItem()`, use `invoice_paid` when invoices exist, fall back to `actual_amount` only when no invoices are linked. Same logic in `budgetItem.model.ts` summary queries.

**Files**: `src/controllers/budgetItem.controller.ts`, `src/models/budgetItem.model.ts`

## Bug 2: Reply Says "Created invoice" for Receipts

**Root cause**: `actionParser.service.ts` maps `forwarded_receipt` → `add_invoice` action type. `replyComposer.service.ts` formats `add_invoice` as "Created invoice".

**Fix**:
- Add `add_receipt` action type alongside `add_invoice`
- Map `forwarded_receipt` emails to `add_receipt` action in actionParser
- Add `add_receipt` case in actionExecutor (reuse invoice creation logic with `source: 'receipt'`)
- Add `add_receipt` formatting in replyComposer ("Added receipt")

**Files**: `actionParser.service.ts`, `actionExecutor.service.ts`, `replyComposer.service.ts`, `classifier.service.ts`

## Bug 3: Correction Detection Broken

**Root cause**: After sending a reply email, the Resend response `id` is discarded. `reply_message_id` column is never populated. When user replies to correct, `findByInReplyTo` returns null → treated as new email.

**Fix**:
- In `email.service.ts`: return `id` from `resend.emails.send()` (Resend returns this as the Message-ID)
- In `replyComposer.service.ts`: capture returned ID, update `email_actions.reply_message_id`
- In `emailAction.model.ts`: verify `findByInReplyTo` query matches correctly (Resend ID format may need `<id@resend.dev>` wrapping to match In-Reply-To header)

**Files**: `src/services/email.service.ts`, `src/services/email/replyComposer.service.ts`, `src/models/emailAction.model.ts`

## Feature: Change Document Type via Email Correction

**Root cause**: `parseCorrectionActions` only handles `update_invoice` and `delete_invoice`. No support for type conversion.

**Fix**:
- Add `convert_to_invoice` and `convert_to_receipt` action types
- In `parseCorrectionActions`, detect phrases like "change to invoice", "treat as invoice", "make it an invoice"
- In `actionExecutor`, implement conversion: update the invoice/document record's type field
- In `replyComposer`, format the conversion confirmation

**Files**: `actionParser.service.ts`, `actionExecutor.service.ts`, `replyComposer.service.ts`

## Execution Order

1. Bug 1 (budget) — independent, fix first
2. Bug 3 (correction detection) — prerequisite for Bug 2 and Feature working correctly
3. Bug 2 (receipt wording) — depends on understanding action types
4. Feature (type conversion) — builds on correction system
5. Commit, deploy, update docs

# Timestamps & Date Handling

Centralized standards for all date/time operations across the application.

---

## Core Principles

1. **Store in UTC** - All database timestamps use `TIMESTAMP WITH TIME ZONE` stored in UTC
2. **Transmit in ISO 8601** - All API requests/responses use ISO 8601 format
3. **Display in user timezone** - Frontend converts to user's local timezone for display
4. **Use date-fns** - Standardize on date-fns library for all date manipulation

---

## Database Standards

### Column Types

```sql
-- Always use TIMESTAMP WITH TIME ZONE for timestamps
created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
deleted_at TIMESTAMP WITH TIME ZONE,

-- Use DATE for date-only fields (no time component)
start_date DATE,
end_date DATE,
due_date DATE,

-- Use TIME for time-only fields (rare)
remind_at_time TIME
```

### Default Values

```sql
-- Current timestamp
DEFAULT NOW()

-- Future date calculation in query
WHERE due_date < CURRENT_DATE
WHERE due_date BETWEEN CURRENT_DATE AND CURRENT_DATE + INTERVAL '7 days'
```

---

## API Format Standards

### Request Format

```typescript
// ISO 8601 for timestamps
{
  "createdAt": "2024-12-23T14:30:00.000Z"
}

// YYYY-MM-DD for dates
{
  "startDate": "2024-12-23",
  "dueDate": "2025-01-15"
}
```

### Response Format

```typescript
// Always include timezone indicator
{
  "createdAt": "2024-12-23T14:30:00.000Z",
  "updatedAt": "2024-12-23T14:30:00.000Z",
  "startDate": "2024-12-23",
  "dueDate": "2025-01-15"
}
```

---

## Backend Implementation

### Location: `src/utils/dates.ts`

```typescript
import { format, parseISO, formatISO, isValid, differenceInDays } from 'date-fns';
import { formatInTimeZone, toZonedTime } from 'date-fns-tz';

// Standard formats
export const DATE_FORMATS = {
  ISO_DATE: 'yyyy-MM-dd',
  ISO_DATETIME: "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'",
  DISPLAY_DATE: 'MMM d, yyyy',
  DISPLAY_DATETIME: 'MMM d, yyyy h:mm a',
  DISPLAY_TIME: 'h:mm a',
} as const;

/**
 * Parse ISO date string to Date object
 */
export function parseDate(dateString: string): Date | null {
  const parsed = parseISO(dateString);
  return isValid(parsed) ? parsed : null;
}

/**
 * Format Date to ISO date string (YYYY-MM-DD)
 */
export function toISODate(date: Date): string {
  return format(date, DATE_FORMATS.ISO_DATE);
}

/**
 * Format Date to ISO datetime string
 */
export function toISODateTime(date: Date): string {
  return formatISO(date);
}

/**
 * Calculate days until a date (negative if past)
 */
export function daysUntil(date: Date): number {
  return differenceInDays(date, new Date());
}

/**
 * Check if a date is overdue
 */
export function isOverdue(date: Date): boolean {
  return differenceInDays(date, new Date()) < 0;
}

/**
 * Get start of today in UTC
 */
export function startOfToday(): Date {
  const now = new Date();
  now.setUTCHours(0, 0, 0, 0);
  return now;
}

/**
 * Format for database query
 */
export function toDBTimestamp(date: Date): string {
  return date.toISOString();
}
```

---

## Frontend Implementation

### Location: `src/lib/dates.ts`

```typescript
import { format, parseISO, formatDistanceToNow, isValid } from 'date-fns';

// Display formats for UI
export const DISPLAY_FORMATS = {
  DATE_SHORT: 'MMM d',           // Dec 23
  DATE_FULL: 'MMM d, yyyy',      // Dec 23, 2024
  DATE_LONG: 'MMMM d, yyyy',     // December 23, 2024
  DATETIME: 'MMM d, yyyy h:mm a', // Dec 23, 2024 2:30 PM
  TIME: 'h:mm a',                // 2:30 PM
  RELATIVE: 'relative',          // 2 days ago
} as const;

/**
 * Format date for display
 */
export function formatDate(
  dateString: string | Date,
  formatType: keyof typeof DISPLAY_FORMATS = 'DATE_FULL'
): string {
  const date = typeof dateString === 'string' ? parseISO(dateString) : dateString;

  if (!isValid(date)) return '';

  if (formatType === 'RELATIVE') {
    return formatDistanceToNow(date, { addSuffix: true });
  }

  return format(date, DISPLAY_FORMATS[formatType]);
}

/**
 * Format for form inputs (YYYY-MM-DD)
 */
export function toInputDate(dateString: string): string {
  const date = parseISO(dateString);
  return isValid(date) ? format(date, 'yyyy-MM-dd') : '';
}

/**
 * Get relative time description
 */
export function getRelativeTime(dateString: string): string {
  const date = parseISO(dateString);
  return formatDistanceToNow(date, { addSuffix: true });
}

/**
 * Check if date is today
 */
export function isToday(dateString: string): boolean {
  const date = parseISO(dateString);
  const today = new Date();
  return format(date, 'yyyy-MM-dd') === format(today, 'yyyy-MM-dd');
}

/**
 * Check if date is in the past
 */
export function isPast(dateString: string): boolean {
  const date = parseISO(dateString);
  return date < new Date();
}

/**
 * Get days until/since date
 */
export function getDaysFromNow(dateString: string): number {
  const date = parseISO(dateString);
  const now = new Date();
  const diffTime = date.getTime() - now.getTime();
  return Math.ceil(diffTime / (1000 * 60 * 60 * 24));
}
```

---

## Display Patterns

### Dates in Tables

| Context | Format | Example |
|---------|--------|---------|
| Created/Updated | Relative | "2 days ago" |
| Due Date (future) | Short | "Dec 23" |
| Due Date (past) | Short + Overdue | "Dec 20 (3 days overdue)" |
| Date Range | Short | "Dec 1 - Dec 15" |

### Dates in Detail Views

| Context | Format | Example |
|---------|--------|---------|
| Created | Full | "December 23, 2024" |
| With Time | DateTime | "Dec 23, 2024 2:30 PM" |
| Milestone | Full | "January 15, 2025" |

---

## Timezone Handling

### User Timezone Storage

```sql
-- In users table
timezone VARCHAR(50) DEFAULT 'America/New_York'
```

### Frontend Timezone Detection

```typescript
// Get user's timezone from browser
const userTimezone = Intl.DateTimeFormat().resolvedOptions().timeZone;

// Send with user profile update
await api.put('/users/me', { timezone: userTimezone });
```

### Displaying in User Timezone

```typescript
import { formatInTimeZone } from 'date-fns-tz';

function formatInUserTz(dateString: string, userTimezone: string): string {
  return formatInTimeZone(
    parseISO(dateString),
    userTimezone,
    'MMM d, yyyy h:mm a zzz'
  );
}
```

---

## Validation

### Zod Schemas

```typescript
import { z } from 'zod';

// Date only (YYYY-MM-DD)
export const dateSchema = z.string().regex(/^\d{4}-\d{2}-\d{2}$/);

// DateTime (ISO 8601)
export const dateTimeSchema = z.string().datetime();

// Optional date
export const optionalDateSchema = dateSchema.optional().nullable();

// Date range validation
export const dateRangeSchema = z.object({
  startDate: dateSchema,
  endDate: dateSchema,
}).refine(
  (data) => new Date(data.startDate) <= new Date(data.endDate),
  { message: 'End date must be after start date' }
);
```

---

## Common Queries

### Overdue Items

```sql
-- Tasks overdue
SELECT * FROM schedule_tasks
WHERE planned_end_date < CURRENT_DATE
AND status != 'completed';

-- Invoices overdue
SELECT * FROM invoices
WHERE due_date < CURRENT_DATE
AND status != 'paid';
```

### Upcoming Items

```sql
-- Milestones in next 7 days
SELECT * FROM schedule_milestones
WHERE target_date BETWEEN CURRENT_DATE AND CURRENT_DATE + INTERVAL '7 days';

-- Reminders due soon
SELECT * FROM reminders
WHERE remind_at <= NOW() + INTERVAL '1 hour'
AND is_dismissed = false;
```

### Date Grouping

```sql
-- Activity by day
SELECT DATE(created_at) as date, COUNT(*)
FROM activity_log
GROUP BY DATE(created_at);

-- Spending by month
SELECT DATE_TRUNC('month', payment_date) as month, SUM(amount)
FROM payments
GROUP BY DATE_TRUNC('month', payment_date);
```

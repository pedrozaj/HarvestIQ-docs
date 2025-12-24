# Validation Patterns

Centralized validation rules and error messages for consistent data integrity across the application.

---

## Validation Library

Use **Zod** for both backend and frontend validation:

```bash
npm install zod
```

---

## Shared Schema Location

```
src/
├── schemas/                 # Shared validation schemas
│   ├── auth.schema.ts
│   ├── project.schema.ts
│   ├── schedule.schema.ts
│   ├── budget.schema.ts
│   ├── document.schema.ts
│   ├── payment.schema.ts
│   ├── task-item.schema.ts
│   ├── notification.schema.ts
│   └── common.schema.ts     # Reusable base schemas
```

---

## Common Base Schemas

### Location: `src/schemas/common.schema.ts`

```typescript
import { z } from 'zod';

// UUID validation
export const uuidSchema = z.string().uuid('Invalid ID format');

// Pagination
export const paginationSchema = z.object({
  limit: z.coerce.number().min(1).max(100).default(20),
  offset: z.coerce.number().min(0).default(0),
});

// Date formats
export const dateSchema = z.string().regex(
  /^\d{4}-\d{2}-\d{2}$/,
  'Date must be in YYYY-MM-DD format'
);

export const dateTimeSchema = z.string().datetime({
  message: 'Invalid datetime format',
});

export const optionalDateSchema = dateSchema.optional().nullable();

// Date range with validation
export const dateRangeSchema = z.object({
  startDate: dateSchema,
  endDate: dateSchema,
}).refine(
  (data) => new Date(data.startDate) <= new Date(data.endDate),
  { message: 'End date must be after start date', path: ['endDate'] }
);

// Money/currency
export const moneySchema = z.coerce
  .number()
  .min(0, 'Amount cannot be negative')
  .multipleOf(0.01, 'Amount must have at most 2 decimal places');

// Percentage
export const percentageSchema = z.coerce
  .number()
  .min(0, 'Percentage cannot be negative')
  .max(100, 'Percentage cannot exceed 100');

// Email
export const emailSchema = z
  .string()
  .email('Invalid email address')
  .toLowerCase()
  .trim();

// Phone (flexible format)
export const phoneSchema = z
  .string()
  .regex(/^[\d\s\-\+\(\)]+$/, 'Invalid phone number format')
  .optional()
  .nullable();

// URL
export const urlSchema = z.string().url('Invalid URL format');

// Non-empty string
export const requiredStringSchema = z
  .string()
  .min(1, 'This field is required')
  .trim();

// Sort order
export const sortOrderSchema = z.enum(['asc', 'desc']).default('desc');
```

---

## Entity Schemas

### Authentication

```typescript
// src/schemas/auth.schema.ts
import { z } from 'zod';

export const loginSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(1, 'Password is required'),
});

export const registerSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain at least one uppercase letter')
    .regex(/[a-z]/, 'Password must contain at least one lowercase letter')
    .regex(/[0-9]/, 'Password must contain at least one number'),
  name: z.string().min(2, 'Name must be at least 2 characters'),
  company_name: z.string().min(2, 'Company name must be at least 2 characters'),
});

export const forgotPasswordSchema = z.object({
  email: z.string().email('Invalid email address'),
});

export const resetPasswordSchema = z.object({
  token: z.string().min(1, 'Reset token is required'),
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain at least one uppercase letter')
    .regex(/[a-z]/, 'Password must contain at least one lowercase letter')
    .regex(/[0-9]/, 'Password must contain at least one number'),
});

export const changePasswordSchema = z.object({
  currentPassword: z.string().min(1, 'Current password is required'),
  newPassword: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain at least one uppercase letter')
    .regex(/[a-z]/, 'Password must contain at least one lowercase letter')
    .regex(/[0-9]/, 'Password must contain at least one number'),
}).refine(
  (data) => data.currentPassword !== data.newPassword,
  { message: 'New password must be different from current password', path: ['newPassword'] }
);
```

### Projects

```typescript
// src/schemas/project.schema.ts
import { z } from 'zod';
import { UNIT_TYPE, PROJECT_STATUS } from '@/constants';

export const createProjectSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters').max(255),
  organizationId: z.string().uuid('Invalid organization'),
  address: z.string().optional().nullable(),
  unitType: z.enum(Object.values(UNIT_TYPE) as [string, ...string[]], {
    errorMap: () => ({ message: 'Invalid unit type' }),
  }),
  units: z.coerce.number().int().min(1, 'Must have at least 1 unit'),
  lotSizeAcres: z.coerce.number().min(0).optional().nullable(),
  startDate: z.string().regex(/^\d{4}-\d{2}-\d{2}$/).optional().nullable(),
  endDate: z.string().regex(/^\d{4}-\d{2}-\d{2}$/).optional().nullable(),
}).refine(
  (data) => !data.startDate || !data.endDate || new Date(data.startDate) <= new Date(data.endDate),
  { message: 'End date must be after start date', path: ['endDate'] }
);

export const updateProjectSchema = createProjectSchema.partial().extend({
  status: z.enum(Object.values(PROJECT_STATUS) as [string, ...string[]]).optional(),
});

export const projectQuerySchema = z.object({
  status: z.enum(Object.values(PROJECT_STATUS) as [string, ...string[]]).optional(),
  organizationId: z.string().uuid().optional(),
  search: z.string().optional(),
  sortBy: z.enum(['name', 'created_at', 'start_date', 'status']).default('created_at'),
  sortOrder: z.enum(['asc', 'desc']).default('desc'),
  limit: z.coerce.number().min(1).max(100).default(20),
  offset: z.coerce.number().min(0).default(0),
});
```

### Schedule Tasks

```typescript
// src/schemas/schedule.schema.ts
import { z } from 'zod';
import { TASK_STATUS, PRIORITY } from '@/constants';

export const createTaskSchema = z.object({
  phaseId: z.string().uuid('Invalid phase'),
  name: z.string().min(1, 'Name is required').max(255),
  description: z.string().optional().nullable(),
  assignedTo: z.string().uuid().optional().nullable(),
  priority: z.enum(Object.values(PRIORITY) as [string, ...string[]]).default('medium'),
  plannedStartDate: z.string().regex(/^\d{4}-\d{2}-\d{2}$/).optional().nullable(),
  plannedEndDate: z.string().regex(/^\d{4}-\d{2}-\d{2}$/).optional().nullable(),
  estimatedHours: z.coerce.number().min(0).optional().nullable(),
}).refine(
  (data) => !data.plannedStartDate || !data.plannedEndDate ||
    new Date(data.plannedStartDate) <= new Date(data.plannedEndDate),
  { message: 'End date must be after start date', path: ['plannedEndDate'] }
);

export const updateTaskSchema = createTaskSchema.partial().extend({
  status: z.enum(Object.values(TASK_STATUS) as [string, ...string[]]).optional(),
  actualStartDate: z.string().regex(/^\d{4}-\d{2}-\d{2}$/).optional().nullable(),
  actualEndDate: z.string().regex(/^\d{4}-\d{2}-\d{2}$/).optional().nullable(),
});

export const addDependencySchema = z.object({
  dependsOnTaskId: z.string().uuid('Invalid task'),
});
```

### Budget Items

```typescript
// src/schemas/budget.schema.ts
import { z } from 'zod';

export const createBudgetItemSchema = z.object({
  categoryId: z.string().uuid('Invalid category'),
  phaseId: z.string().uuid().optional().nullable(),
  name: z.string().min(1, 'Name is required').max(255),
  description: z.string().optional().nullable(),
  estimatedAmount: z.coerce.number().min(0, 'Amount cannot be negative'),
  actualAmount: z.coerce.number().min(0).optional().nullable(),
});

export const updateBudgetItemSchema = createBudgetItemSchema.partial();

export const createCategorySchema = z.object({
  name: z.string().min(1, 'Name is required').max(100),
});
```

---

## Error Messages

### Centralized Error Messages

```typescript
// src/lib/validation-messages.ts
export const VALIDATION_MESSAGES = {
  // Field-level
  required: 'This field is required',
  email: 'Please enter a valid email address',
  minLength: (min: number) => `Must be at least ${min} characters`,
  maxLength: (max: number) => `Must be no more than ${max} characters`,
  minValue: (min: number) => `Must be at least ${min}`,
  maxValue: (max: number) => `Must be no more than ${max}`,
  invalidFormat: 'Invalid format',
  invalidDate: 'Please enter a valid date',
  invalidUuid: 'Invalid selection',

  // Password
  passwordMinLength: 'Password must be at least 8 characters',
  passwordUppercase: 'Password must contain an uppercase letter',
  passwordLowercase: 'Password must contain a lowercase letter',
  passwordNumber: 'Password must contain a number',
  passwordMismatch: 'Passwords do not match',
  passwordSameAsCurrent: 'New password must be different from current',

  // Date ranges
  endDateBeforeStart: 'End date must be after start date',
  dateInPast: 'Date cannot be in the past',

  // Business rules
  budgetExceeded: 'This would exceed the budget',
  duplicateEmail: 'This email is already registered',
  circularDependency: 'This would create a circular dependency',
  selfAssignment: 'Cannot assign to itself',

  // File upload
  fileTooLarge: (maxMb: number) => `File must be smaller than ${maxMb}MB`,
  invalidFileType: 'This file type is not supported',
};
```

---

## Backend Validation Middleware

```typescript
// src/middleware/validate.ts
import { Request, Response, NextFunction } from 'express';
import { ZodSchema, ZodError } from 'zod';

export function validate(schema: ZodSchema, source: 'body' | 'query' | 'params' = 'body') {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      req[source] = await schema.parseAsync(req[source]);
      next();
    } catch (error) {
      if (error instanceof ZodError) {
        return res.status(400).json({
          error: {
            code: 'VAL_3001',
            message: 'Validation failed',
            details: error.errors.map((e) => ({
              field: e.path.join('.'),
              message: e.message,
            })),
          },
        });
      }
      next(error);
    }
  };
}

// Usage in routes
router.post('/projects',
  requireAuth,
  validate(createProjectSchema),
  projectController.create
);
```

---

## Frontend Form Validation

### React Hook Form + Zod

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { createProjectSchema } from '@/schemas/project.schema';

function CreateProjectForm() {
  const form = useForm({
    resolver: zodResolver(createProjectSchema),
    defaultValues: {
      name: '',
      unitType: 'single_family',
      units: 1,
    },
  });

  const onSubmit = async (data: z.infer<typeof createProjectSchema>) => {
    // data is fully typed and validated
  };

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)}>
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Project Name</FormLabel>
              <FormControl>
                <Input {...field} />
              </FormControl>
              <FormMessage /> {/* Displays error automatically */}
            </FormItem>
          )}
        />
      </form>
    </Form>
  );
}
```

---

## File Upload Validation

```typescript
// src/schemas/document.schema.ts
export const ALLOWED_FILE_TYPES = {
  document: ['application/pdf', 'image/jpeg', 'image/png', 'image/webp'],
  image: ['image/jpeg', 'image/png', 'image/webp', 'image/gif'],
};

export const MAX_FILE_SIZE = {
  document: 10 * 1024 * 1024,  // 10MB
  image: 5 * 1024 * 1024,      // 5MB
};

export const fileUploadSchema = z.object({
  name: z.string().min(1),
  type: z.enum(['invoice', 'estimate', 'contract', 'permit', 'photo', 'other']),
  mimeType: z.string().refine(
    (type) => ALLOWED_FILE_TYPES.document.includes(type),
    'File type not supported'
  ),
  size: z.number().max(MAX_FILE_SIZE.document, 'File too large (max 10MB)'),
});

// Frontend validation before upload
function validateFile(file: File): string | null {
  if (!ALLOWED_FILE_TYPES.document.includes(file.type)) {
    return 'File type not supported. Please upload PDF or image files.';
  }
  if (file.size > MAX_FILE_SIZE.document) {
    return 'File too large. Maximum size is 10MB.';
  }
  return null;
}
```

---

## API Error Response Format

```typescript
// Standard validation error response
{
  "error": {
    "code": "VAL_3001",
    "message": "Validation failed",
    "details": [
      { "field": "email", "message": "Invalid email address" },
      { "field": "password", "message": "Password must be at least 8 characters" }
    ]
  }
}

// Frontend error handling
async function handleApiError(response: Response) {
  const data = await response.json();

  if (data.error?.details) {
    // Set form errors from API response
    data.error.details.forEach(({ field, message }) => {
      form.setError(field, { message });
    });
  } else {
    // Show general error toast
    toast.error(data.error?.message || 'An error occurred');
  }
}
```

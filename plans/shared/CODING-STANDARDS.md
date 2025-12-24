# Coding Standards

Conventions and patterns for consistent, maintainable code across the application.

---

## Project Structure

### Backend (`HarvestIQ-backend/`)

```
src/
├── config/              # Environment, database config
│   ├── database.ts
│   └── env.ts
├── constants/           # Shared constants (see CONSTANTS.md)
│   └── index.ts
├── controllers/         # Route handlers
│   ├── auth.controller.ts
│   ├── project.controller.ts
│   └── ...
├── middleware/          # Express middleware
│   ├── auth.ts
│   ├── validate.ts
│   ├── errorHandler.ts
│   └── rateLimiter.ts
├── models/              # Database queries and business logic
│   ├── user.model.ts
│   ├── project.model.ts
│   └── ...
├── routes/              # Route definitions
│   ├── index.ts
│   ├── auth.routes.ts
│   ├── project.routes.ts
│   └── ...
├── schemas/             # Zod validation schemas
│   ├── auth.schema.ts
│   ├── project.schema.ts
│   └── ...
├── services/            # External services, business logic
│   ├── email.service.ts
│   ├── storage.service.ts
│   └── ai.service.ts
├── jobs/                # Background job handlers
│   ├── reminder.job.ts
│   ├── notification.job.ts
│   └── ...
├── utils/               # Helper functions
│   ├── dates.ts
│   ├── crypto.ts
│   └── ...
├── types/               # TypeScript type definitions
│   └── index.ts
└── app.ts               # Express app setup
```

### Frontend (`HarvestIQ/`)

```
src/
├── app/                 # Next.js app router pages
│   ├── (auth)/          # Auth layout group
│   │   ├── login/
│   │   └── register/
│   ├── (dashboard)/     # Main app layout group
│   │   ├── dashboard/
│   │   ├── projects/
│   │   └── ...
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── ui/              # shadcn/ui components
│   ├── dialogs/         # Modal components
│   ├── forms/           # Form components
│   ├── layout/          # Layout components (sidebar, header)
│   └── [feature]/       # Feature-specific components
├── hooks/               # Custom React hooks
│   ├── useAuth.ts
│   ├── useProjects.ts
│   └── ...
├── lib/                 # Utilities and helpers
│   ├── api.ts           # API client
│   ├── dates.ts
│   ├── utils.ts
│   └── constants.ts
├── schemas/             # Zod schemas (shared with backend)
├── stores/              # State management (if using)
└── types/               # TypeScript types
```

---

## Naming Conventions

### Files and Directories

| Type | Convention | Example |
|------|------------|---------|
| React components | PascalCase | `ProjectCard.tsx` |
| Hooks | camelCase with `use` prefix | `useProjects.ts` |
| Utilities | camelCase | `formatDate.ts` |
| Constants | camelCase file, SCREAMING_SNAKE vars | `constants.ts` |
| Types | PascalCase | `Project.ts` |
| Schemas | camelCase with `.schema` suffix | `project.schema.ts` |
| API routes | kebab-case | `project.routes.ts` |
| Controllers | camelCase with `.controller` suffix | `project.controller.ts` |

### Variables and Functions

| Type | Convention | Example |
|------|------------|---------|
| Variables | camelCase | `projectCount` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_FILE_SIZE` |
| Functions | camelCase | `getProjectById` |
| React components | PascalCase | `ProjectCard` |
| Boolean variables | `is`, `has`, `can` prefix | `isLoading`, `hasError` |
| Event handlers | `handle` or `on` prefix | `handleSubmit`, `onClose` |
| Async functions | verb form | `fetchProjects`, `createProject` |

### Database

| Type | Convention | Example |
|------|------------|---------|
| Tables | snake_case, plural | `schedule_tasks` |
| Columns | snake_case | `created_at` |
| Foreign keys | singular_table_id | `project_id` |
| Indexes | idx_table_column | `idx_projects_builder_id` |
| Enums | snake_case | `project_status` |

### API

| Type | Convention | Example |
|------|------------|---------|
| Endpoints | kebab-case, plural nouns | `/api/projects` |
| Query params | snake_case | `?project_id=X` |
| Request body | camelCase | `{ projectId: "..." }` |
| Response body | camelCase | `{ createdAt: "..." }` |

---

## TypeScript Guidelines

### Type Definitions

```typescript
// Prefer interfaces for object shapes
interface Project {
  id: string;
  name: string;
  status: ProjectStatus;
  createdAt: string;
}

// Use type for unions, intersections, primitives
type ProjectStatus = 'planning' | 'active' | 'on_hold' | 'completed';
type ProjectWithBudget = Project & { budget: Budget };

// Use const assertions for literal types
const STATUS = {
  ACTIVE: 'active',
  COMPLETED: 'completed',
} as const;
```

### Function Types

```typescript
// Explicit return types for public functions
function getProject(id: string): Promise<Project> {
  // ...
}

// Arrow functions for callbacks
const handleClick = (e: React.MouseEvent) => {
  // ...
};

// Generic functions when needed
function findById<T extends { id: string }>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id);
}
```

### Avoid `any`

```typescript
// Bad
function processData(data: any) { }

// Good
function processData(data: unknown) {
  if (isProject(data)) {
    // data is now typed as Project
  }
}

// Type guards
function isProject(data: unknown): data is Project {
  return typeof data === 'object' && data !== null && 'id' in data;
}
```

---

## React Patterns

### Component Structure

```typescript
// Imports (external, internal, types, styles)
import { useState } from 'react';
import { Button } from '@/components/ui/button';
import type { Project } from '@/types';

// Types/interfaces
interface ProjectCardProps {
  project: Project;
  onEdit: (id: string) => void;
}

// Component
export function ProjectCard({ project, onEdit }: ProjectCardProps) {
  // Hooks first
  const [isExpanded, setIsExpanded] = useState(false);

  // Handlers
  const handleEdit = () => {
    onEdit(project.id);
  };

  // Early returns for loading/error states
  if (!project) return null;

  // Render
  return (
    <div>
      {/* JSX */}
    </div>
  );
}
```

### Custom Hooks

```typescript
// hooks/useProjects.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { api } from '@/lib/api';
import type { Project, CreateProjectInput } from '@/types';

export function useProjects(params?: { status?: string }) {
  return useQuery({
    queryKey: ['projects', params],
    queryFn: () => api.get<Project[]>('/projects', { params }),
  });
}

export function useCreateProject() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateProjectInput) => api.post<Project>('/projects', data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['projects'] });
    },
  });
}
```

### Error Boundaries

```typescript
// components/ErrorBoundary.tsx
'use client';

import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
}

export class ErrorBoundary extends Component<Props, State> {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || <div>Something went wrong</div>;
    }
    return this.props.children;
  }
}
```

---

## API Design Patterns

### Controller Pattern

```typescript
// controllers/project.controller.ts
import { Request, Response, NextFunction } from 'express';
import * as projectModel from '@/models/project.model';

export async function getAll(req: Request, res: Response, next: NextFunction) {
  try {
    const { builderId } = req.user!;
    const projects = await projectModel.findByBuilder(builderId, req.query);
    res.json({ data: projects });
  } catch (error) {
    next(error);
  }
}

export async function getById(req: Request, res: Response, next: NextFunction) {
  try {
    const { id } = req.params;
    const { builderId } = req.user!;

    const project = await projectModel.findById(id, builderId);
    if (!project) {
      return res.status(404).json({ error: { code: 'RES_4001', message: 'Project not found' } });
    }

    res.json({ data: project });
  } catch (error) {
    next(error);
  }
}
```

### Model Pattern

```typescript
// models/project.model.ts
import { db } from '@/config/database';
import type { Project, CreateProjectInput } from '@/types';

export async function findByBuilder(
  builderId: string,
  options: { limit?: number; offset?: number; status?: string }
): Promise<Project[]> {
  const { limit = 20, offset = 0, status } = options;

  const result = await db.query(
    `SELECT * FROM projects
     WHERE builder_id = $1
     AND deleted_at IS NULL
     ${status ? 'AND status = $4' : ''}
     ORDER BY created_at DESC
     LIMIT $2 OFFSET $3`,
    status ? [builderId, limit, offset, status] : [builderId, limit, offset]
  );

  return result.rows;
}

export async function create(builderId: string, data: CreateProjectInput): Promise<Project> {
  const result = await db.query(
    `INSERT INTO projects (builder_id, organization_id, name, unit_type, units, ...)
     VALUES ($1, $2, $3, $4, $5, ...)
     RETURNING *`,
    [builderId, data.organizationId, data.name, data.unitType, data.units, ...]
  );

  return result.rows[0];
}
```

### Error Handling

```typescript
// middleware/errorHandler.ts
import { Request, Response, NextFunction } from 'express';
import { ZodError } from 'zod';

export class AppError extends Error {
  constructor(
    public statusCode: number,
    public code: string,
    message: string
  ) {
    super(message);
  }
}

export function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  console.error(err);

  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: { code: err.code, message: err.message },
    });
  }

  if (err instanceof ZodError) {
    return res.status(400).json({
      error: {
        code: 'VAL_3001',
        message: 'Validation failed',
        details: err.errors,
      },
    });
  }

  res.status(500).json({
    error: { code: 'SYS_9001', message: 'Internal server error' },
  });
}
```

---

## Database Patterns

### Query Builder vs Raw SQL

```typescript
// Use parameterized queries always
const result = await db.query(
  'SELECT * FROM projects WHERE id = $1 AND builder_id = $2',
  [id, builderId]
);

// Never string concatenation
// BAD: `SELECT * FROM projects WHERE id = '${id}'`
```

### Transactions

```typescript
export async function createProjectWithBudget(data: CreateProjectWithBudgetInput) {
  const client = await db.pool.connect();

  try {
    await client.query('BEGIN');

    const project = await client.query(
      'INSERT INTO projects (...) VALUES (...) RETURNING *',
      [...]
    );

    await client.query(
      'INSERT INTO budget_items (...) VALUES (...)',
      [...]
    );

    await client.query('COMMIT');
    return project.rows[0];
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
```

### Soft Deletes

```typescript
// All deletions are soft deletes
export async function remove(id: string, builderId: string): Promise<boolean> {
  const result = await db.query(
    `UPDATE projects SET deleted_at = NOW()
     WHERE id = $1 AND builder_id = $2 AND deleted_at IS NULL`,
    [id, builderId]
  );
  return result.rowCount > 0;
}

// All queries exclude soft-deleted records
export async function findById(id: string, builderId: string): Promise<Project | null> {
  const result = await db.query(
    `SELECT * FROM projects
     WHERE id = $1 AND builder_id = $2 AND deleted_at IS NULL`,
    [id, builderId]
  );
  return result.rows[0] || null;
}
```

---

## Testing Standards

### Test File Location

```
src/
├── controllers/
│   ├── project.controller.ts
│   └── __tests__/
│       └── project.controller.test.ts
```

### Test Naming

```typescript
describe('ProjectController', () => {
  describe('getById', () => {
    it('returns project when found', async () => { });
    it('returns 404 when project not found', async () => { });
    it('returns 403 when user lacks access', async () => { });
  });
});
```

### Test Data

```typescript
// Use factories for test data
const createTestProject = (overrides?: Partial<Project>): Project => ({
  id: 'test-id',
  name: 'Test Project',
  status: 'active',
  ...overrides,
});
```

---

## Git Conventions

### Branch Naming

```
feature/add-project-timeline
bugfix/fix-payment-calculation
hotfix/security-patch
chore/update-dependencies
```

### Commit Messages

```
feat: add project timeline view
fix: correct payment calculation rounding
docs: update API documentation
refactor: extract date utilities
test: add project controller tests
chore: update dependencies
```

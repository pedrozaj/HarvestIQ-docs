# Phase 3: API Endpoints

Schedule and budget management endpoints.

---

## Authorization

| Resource | Read | Create/Update | Delete |
|----------|------|---------------|--------|
| Budget Categories | All | Admin | Admin |
| Schedule Phases | All | Admin/Manager | Admin/Manager |
| Schedule Tasks | All | Admin/Manager/Member | Admin/Manager |
| Schedule Milestones | All | Admin/Manager | Admin/Manager |
| Budget Items | All | Admin/Manager | Admin/Manager |

> **Note:** All DELETE operations are soft deletes (set `deleted_at` timestamp).

---

## Budget Categories

### GET /budget-categories

List builder's budget categories.

```typescript
Response: {
  data: [{
    id: string;
    name: string;
    isActive: boolean;
  }]
}
```

### POST /budget-categories

Create category.

```typescript
Request: { name: string }
Response: { data: { id, name, isActive, createdAt } }
```

### PUT /budget-categories/:id

Update category.

```typescript
Request: { name?: string; isActive?: boolean }
Response: { data: { id, name, isActive, updatedAt } }
```

### DELETE /budget-categories/:id

Deactivate category (soft delete via isActive=false).

---

## Schedule Phases

### GET /projects/:id/phases

List project phases.

```typescript
Response: {
  data: [{
    id: string;
    name: string;
    description: string | null;
    sortOrder: number;
    status: string;
    startDate: string | null;
    endDate: string | null;
    taskCount: number;
    completedTaskCount: number;
  }]
}
```

### POST /projects/:id/phases

Create phase.

```typescript
Request: {
  name: string;
  description?: string;
  startDate?: string;
  endDate?: string;
}
Response: { data: { id, name, sortOrder, status, ... } }
```

### PUT /projects/:id/phases/:phaseId

Update phase.

### DELETE /projects/:id/phases/:phaseId

Delete phase (cascades to tasks).

### PUT /projects/:id/phases/reorder

Reorder phases.

```typescript
Request: {
  phases: [{ id: string; sortOrder: number }]
}
```

---

## Schedule Tasks

### GET /projects/:id/tasks

List project tasks with filters.

```typescript
Query: {
  phaseId?: string;
  status?: string;
  assignedTo?: string;
  priority?: string;
  limit?: number;
  offset?: number;
}

Response: {
  data: [{
    id: string;
    name: string;
    phase: { id, name };
    assignedTo: { id, name } | null;
    status: string;
    priority: string;
    plannedStartDate: string | null;
    plannedEndDate: string | null;
    actualStartDate: string | null;
    actualEndDate: string | null;
    dependencies: string[];  // task IDs
  }],
  total: number
}
```

### POST /projects/:id/tasks

Create task.

```typescript
Request: {
  phaseId: string;
  name: string;
  description?: string;
  assignedTo?: string;
  priority?: string;
  plannedStartDate?: string;
  plannedEndDate?: string;
  estimatedHours?: number;
}
```

### GET /projects/:id/tasks/:taskId

Get task details.

### PUT /projects/:id/tasks/:taskId

Update task.

### DELETE /projects/:id/tasks/:taskId

Delete task.

### POST /projects/:id/tasks/:taskId/dependencies

Add dependency.

```typescript
Request: { dependsOnTaskId: string }
```

**Validation:** Check for circular dependencies before insert.

### DELETE /projects/:id/tasks/:taskId/dependencies/:depId

Remove dependency.

---

## Schedule Milestones

### GET /projects/:id/milestones

List milestones.

```typescript
Response: {
  data: [{
    id: string;
    name: string;
    phase: { id, name } | null;
    targetDate: string;
    actualDate: string | null;
    status: string;
  }]
}
```

### POST /projects/:id/milestones

Create milestone.

```typescript
Request: {
  name: string;
  phaseId?: string;
  targetDate: string;
  description?: string;
}
```

### PUT /projects/:id/milestones/:id

Update milestone.

### DELETE /projects/:id/milestones/:id

Delete milestone.

---

## Timeline

### GET /projects/:id/timeline

Get aggregated timeline data for Gantt chart.

```typescript
Response: {
  data: {
    phases: [{
      id: string;
      name: string;
      sortOrder: number;
      tasks: [{
        id: string;
        name: string;
        status: string;
        priority: string;
        assignedTo: { id, name } | null;
        plannedStart: string | null;
        plannedEnd: string | null;
        actualStart: string | null;
        actualEnd: string | null;
        dependencies: string[];
      }];
    }];
    milestones: [{
      id: string;
      name: string;
      targetDate: string;
      status: string;
    }];
    projectStart: string | null;
    projectEnd: string | null;
  }
}
```

---

## Budget Items

### GET /projects/:id/budget

List budget items.

```typescript
Query: {
  categoryId?: string;
  phaseId?: string;
  limit?: number;
  offset?: number;
}

Response: {
  data: [{
    id: string;
    name: string;
    category: { id, name };
    phase: { id, name } | null;
    estimatedAmount: number;
    actualAmount: number;
    variance: number;
  }],
  total: number
}
```

### POST /projects/:id/budget

Create budget item.

```typescript
Request: {
  categoryId: string;
  phaseId?: string;
  name: string;
  description?: string;
  estimatedAmount: number;
  actualAmount?: number;
}
```

### PUT /projects/:id/budget/:itemId

Update budget item.

### DELETE /projects/:id/budget/:itemId

Delete budget item.

### GET /projects/:id/budget/summary

Get budget summary with totals.

```typescript
Response: {
  data: {
    totalEstimated: number;
    totalActual: number;
    variance: number;
    variancePercent: number;
    costPerUnit: number;
    byCategory: [{
      id: string;
      name: string;
      estimated: number;
      actual: number;
      variance: number;
    }];
    byPhase: [{
      id: string;
      name: string;
      estimated: number;
      actual: number;
    }];
  }
}
```

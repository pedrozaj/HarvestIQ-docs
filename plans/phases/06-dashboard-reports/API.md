# Phase 6: API Endpoints

Dashboard, search, and report endpoints.

---

## Dashboard

### GET /dashboard

Get aggregated dashboard data for current user.

```typescript
Response: {
  data: {
    summary: {
      activeProjects: number;
      totalBudget: number;
      totalSpent: number;
      overdueTasks: number;
      upcomingMilestones: number;
      unpaidInvoices: number;
      unpaidAmount: number;
    };
    projects: [{
      id: string;
      name: string;
      unitType: string;
      units: number;
      status: string;
      budgetProgress: number;       // percentage
      scheduleProgress: number;     // percentage
      nextMilestone: {
        name: string;
        dueDate: string;
        daysUntil: number;
      } | null;
    }];
    myTaskItems: [{
      id: string;
      title: string;
      project: { id, name } | null;
      dueDate: string | null;
      isOverdue: boolean;
      priority: string;
    }];
    upcomingReminders: [{
      id: string;
      title: string;
      remindAt: string;
    }];
    recentActivity: [{
      id: string;
      user: { id, name };
      project: { id, name } | null;
      action: string;
      entityType: string;
      createdAt: string;
    }];
    insights: [{
      id: string;
      type: string;
      title: string;
      description: string;
      severity: string;
      projectId: string | null;
    }];
  }
}
```

---

## Global Search

### GET /search

Search across all entity types.

```typescript
Query: {
  q: string;          // Search query (min 2 chars)
  types?: string;     // Comma-separated: projects,tasks,documents,invoices
  limit?: number;     // Per type limit, default 5
}

Response: {
  data: {
    projects: [{
      id: string;
      name: string;
      address: string | null;
      status: string;
    }];
    tasks: [{
      id: string;
      name: string;
      project: { id, name };
      status: string;
    }];
    documents: [{
      id: string;
      name: string;
      project: { id, name };
      type: string;
    }];
    invoices: [{
      id: string;
      invoiceNumber: string;
      project: { id, name };
      amount: number;
      status: string;
    }];
    taskItems: [{
      id: string;
      title: string;
      project: { id, name } | null;
    }];
  }
}
```

---

## Reports

### GET /projects/:id/reports/schedule

Schedule variance report.

```typescript
Response: {
  data: {
    summary: {
      totalTasks: number;
      completedTasks: number;
      overdueTasks: number;
      blockedTasks: number;
      onTimeCompletion: number;  // percentage
    };
    phases: [{
      id: string;
      name: string;
      status: string;
      plannedStart: string | null;
      plannedEnd: string | null;
      actualStart: string | null;
      actualEnd: string | null;
      variance: number;  // days difference
      tasks: [{
        id: string;
        name: string;
        plannedEnd: string | null;
        actualEnd: string | null;
        variance: number;
        status: string;
      }];
    }];
    milestones: [{
      id: string;
      name: string;
      targetDate: string;
      actualDate: string | null;
      status: string;
      variance: number;
    }];
  }
}
```

### GET /projects/:id/reports/budget

Budget breakdown report.

```typescript
Response: {
  data: {
    summary: {
      totalEstimated: number;
      totalActual: number;
      variance: number;
      variancePercent: number;
      costPerUnit: number;
    };
    byCategory: [{
      id: string;
      name: string;
      estimated: number;
      actual: number;
      variance: number;
      variancePercent: number;
      percentOfTotal: number;
    }];
    byPhase: [{
      id: string;
      name: string;
      estimated: number;
      actual: number;
    }];
    monthlySpending: [{
      month: string;       // YYYY-MM
      amount: number;
    }];
  }
}
```

### GET /reports/spending

Cross-project spending trends.

```typescript
Query: {
  startDate?: string;
  endDate?: string;
  projectIds?: string;  // Comma-separated
}

Response: {
  data: {
    totalSpent: number;
    byProject: [{
      id: string;
      name: string;
      spent: number;
      percentOfTotal: number;
    }];
    byCategory: [{
      id: string;
      name: string;
      spent: number;
      percentOfTotal: number;
    }];
    byMonth: [{
      month: string;
      amount: number;
    }];
    efficiency: [{
      projectId: string;
      projectName: string;
      costPerUnit: number;
      budgetVariance: number;
    }];
  }
}
```

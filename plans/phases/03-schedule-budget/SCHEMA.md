# Phase 3: Database Schema

Schedule and budget tables.

---

## Migration Order

```
007_create_budget_categories.sql
008_create_schedule_phases.sql
009_create_schedule_milestones.sql
010_create_schedule_tasks.sql
011_create_task_dependencies.sql
012_create_budget_items.sql
```

---

## Table: budget_categories

Builder-level cost categories.

```sql
CREATE TABLE budget_categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  name VARCHAR(100) NOT NULL,
  is_active BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  CONSTRAINT uq_budget_categories_name UNIQUE (builder_id, name)
);

CREATE INDEX idx_budget_categories_builder_id ON budget_categories(builder_id);
```

**Default Categories:** Permits, Site Work, Foundation, Framing, Roofing, Electrical, Plumbing, HVAC, Insulation, Drywall, Flooring, Painting, Cabinets, Countertops, Appliances, Fixtures, Landscaping, Cleanup, Contingency

---

## Table: schedule_phases

Project phases for grouping tasks.

```sql
CREATE TABLE schedule_phases (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  sort_order INTEGER NOT NULL DEFAULT 0,
  status VARCHAR(50) NOT NULL DEFAULT 'not_started',
    -- values: not_started, in_progress, completed
  start_date DATE,
  end_date DATE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  CONSTRAINT fk_phases_project_same_builder CHECK (...)
);

CREATE INDEX idx_schedule_phases_builder_id ON schedule_phases(builder_id);
CREATE INDEX idx_schedule_phases_project_id ON schedule_phases(project_id);
```

---

## Table: schedule_milestones

Key project dates to track.

```sql
CREATE TABLE schedule_milestones (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  phase_id UUID REFERENCES schedule_phases(id) ON DELETE SET NULL,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  target_date DATE NOT NULL,
  actual_date DATE,
  status VARCHAR(50) NOT NULL DEFAULT 'upcoming',
    -- values: upcoming, achieved, missed
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_schedule_milestones_project_id ON schedule_milestones(project_id);
CREATE INDEX idx_schedule_milestones_target_date ON schedule_milestones(target_date);
```

---

## Table: schedule_tasks

Individual work items.

```sql
CREATE TABLE schedule_tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  phase_id UUID NOT NULL REFERENCES schedule_phases(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  assigned_to UUID REFERENCES users(id) ON DELETE SET NULL,
  status VARCHAR(50) NOT NULL DEFAULT 'not_started',
    -- values: not_started, in_progress, completed, blocked
  priority VARCHAR(20) NOT NULL DEFAULT 'medium',
    -- values: low, medium, high, urgent
  planned_start_date DATE,
  planned_end_date DATE,
  actual_start_date DATE,
  actual_end_date DATE,
  estimated_hours DECIMAL(10,2),
  actual_hours DECIMAL(10,2),
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_schedule_tasks_project_id ON schedule_tasks(project_id);
CREATE INDEX idx_schedule_tasks_phase_id ON schedule_tasks(phase_id);
CREATE INDEX idx_schedule_tasks_assigned_to ON schedule_tasks(assigned_to);
CREATE INDEX idx_schedule_tasks_status ON schedule_tasks(status);
```

---

## Table: task_dependencies

Links between dependent tasks.

```sql
CREATE TABLE task_dependencies (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  task_id UUID NOT NULL REFERENCES schedule_tasks(id) ON DELETE CASCADE,
  depends_on_task_id UUID NOT NULL REFERENCES schedule_tasks(id) ON DELETE CASCADE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  CONSTRAINT uq_task_dependencies UNIQUE (task_id, depends_on_task_id),
  CONSTRAINT chk_no_self_dependency CHECK (task_id != depends_on_task_id)
);

CREATE INDEX idx_task_dependencies_task_id ON task_dependencies(task_id);
CREATE INDEX idx_task_dependencies_depends_on ON task_dependencies(depends_on_task_id);
```

**Circular Dependency Check:** Implement in application layer using recursive CTE or graph traversal before insert.

---

## Table: budget_items

Individual budget line items.

```sql
CREATE TABLE budget_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  category_id UUID NOT NULL REFERENCES budget_categories(id),
  phase_id UUID REFERENCES schedule_phases(id) ON DELETE SET NULL,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  estimated_amount DECIMAL(12,2) NOT NULL DEFAULT 0,
  actual_amount DECIMAL(12,2) DEFAULT 0,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_budget_items_project_id ON budget_items(project_id);
CREATE INDEX idx_budget_items_category_id ON budget_items(category_id);
CREATE INDEX idx_budget_items_phase_id ON budget_items(phase_id);
```

---

## Key Queries

### Timeline Query (for Gantt)

```sql
SELECT
  p.id as phase_id, p.name as phase_name, p.sort_order,
  t.id as task_id, t.name as task_name,
  t.planned_start_date, t.planned_end_date,
  t.actual_start_date, t.actual_end_date,
  t.status, t.priority,
  u.name as assigned_to_name,
  array_agg(td.depends_on_task_id) FILTER (WHERE td.id IS NOT NULL) as dependencies
FROM schedule_phases p
LEFT JOIN schedule_tasks t ON t.phase_id = p.id
LEFT JOIN users u ON t.assigned_to = u.id
LEFT JOIN task_dependencies td ON td.task_id = t.id
WHERE p.project_id = $1 AND p.builder_id = $2
GROUP BY p.id, t.id, u.name
ORDER BY p.sort_order, t.planned_start_date;
```

### Budget Summary

```sql
SELECT
  bc.id as category_id, bc.name as category_name,
  COALESCE(SUM(bi.estimated_amount), 0) as estimated,
  COALESCE(SUM(bi.actual_amount), 0) as actual
FROM budget_categories bc
LEFT JOIN budget_items bi ON bi.category_id = bc.id AND bi.project_id = $1
WHERE bc.builder_id = $2 AND bc.is_active = true
GROUP BY bc.id
ORDER BY bc.name;
```

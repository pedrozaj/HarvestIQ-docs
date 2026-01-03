# Phase 8: Capital Risk Schema

## Property Valuation Fields (projects table)

Added to the existing `projects` table:

```sql
-- Migration: 046_add_property_valuation_fields
ALTER TABLE projects
ADD COLUMN IF NOT EXISTS purchase_price DECIMAL(14,2),
ADD COLUMN IF NOT EXISTS appraised_value DECIMAL(14,2),
ADD COLUMN IF NOT EXISTS target_arv DECIMAL(14,2),
ADD COLUMN IF NOT EXISTS valuation_date DATE,
ADD COLUMN IF NOT EXISTS appraisal_document_id UUID REFERENCES documents(id);

-- Migration: 047_add_predicted_arv
ALTER TABLE projects
ADD COLUMN IF NOT EXISTS predicted_arv DECIMAL(14,2);

-- Column documentation
COMMENT ON COLUMN projects.purchase_price IS 'Total land/property purchase price (project-level, NOT per-unit)';
COMMENT ON COLUMN projects.appraised_value IS 'Current appraised value per unit';
COMMENT ON COLUMN projects.target_arv IS 'Target After Repair Value per unit (original goal)';
COMMENT ON COLUMN projects.predicted_arv IS 'AI-predicted ARV per unit at project completion';
COMMENT ON COLUMN projects.valuation_date IS 'Date of most recent appraisal';
COMMENT ON COLUMN projects.appraisal_document_id IS 'Reference to uploaded appraisal document';
```

**Important:** `purchase_price` is the total land cost (project-level), while `appraised_value`, `target_arv`, and `predicted_arv` are **per-unit** values that get multiplied by `units` for project totals.

---

## Project Risk Metrics

```sql
CREATE TABLE project_risk_metrics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,

  -- Schedule risk factors
  overdue_tasks_count INTEGER NOT NULL DEFAULT 0,
  blocked_tasks_count INTEGER NOT NULL DEFAULT 0,
  phase_delay_days INTEGER NOT NULL DEFAULT 0,
  milestone_slippage_days INTEGER NOT NULL DEFAULT 0,
  schedule_risk_score DECIMAL(5,2) NOT NULL DEFAULT 0,

  -- Budget risk factors
  budget_variance_percent DECIMAL(8,2) NOT NULL DEFAULT 0,
  over_budget_categories_count INTEGER NOT NULL DEFAULT 0,
  burn_rate_deviation DECIMAL(8,2) NOT NULL DEFAULT 0,
  cost_per_unit_variance DECIMAL(8,2) NOT NULL DEFAULT 0,
  budget_risk_score DECIMAL(5,2) NOT NULL DEFAULT 0,

  -- Composite metrics
  composite_risk_score DECIMAL(5,2) NOT NULL DEFAULT 0,
  risk_level VARCHAR(20) NOT NULL DEFAULT 'low',
    -- values: low, medium, high, critical
  capital_at_risk DECIMAL(14,2) NOT NULL DEFAULT 0,

  -- Trend tracking
  trend VARCHAR(20) NOT NULL DEFAULT 'stable',
    -- values: improving, stable, worsening
  previous_composite_score DECIMAL(5,2),
  trend_velocity DECIMAL(5,2) NOT NULL DEFAULT 0,

  -- Timestamps
  calculated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  CONSTRAINT unique_project_risk_metrics UNIQUE(project_id),
  CONSTRAINT fk_risk_metrics_builder CHECK (
    EXISTS (SELECT 1 FROM projects WHERE id = project_id AND builder_id = project_risk_metrics.builder_id)
  )
);

CREATE INDEX idx_project_risk_metrics_builder_id ON project_risk_metrics(builder_id);
CREATE INDEX idx_project_risk_metrics_project_id ON project_risk_metrics(project_id);
CREATE INDEX idx_project_risk_metrics_risk_level ON project_risk_metrics(risk_level);
CREATE INDEX idx_project_risk_metrics_composite_score ON project_risk_metrics(composite_risk_score DESC);
```

---

## Risk Interventions

```sql
CREATE TABLE risk_interventions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,

  -- Intervention details
  intervention_type VARCHAR(50) NOT NULL,
    -- values: overdue_task, blocked_task, phase_delay, milestone_slip,
    --         over_budget, burn_rate, cost_variance
  severity VARCHAR(20) NOT NULL,
    -- values: low, medium, high, critical
  priority_rank INTEGER NOT NULL DEFAULT 1,

  -- Content
  title VARCHAR(255) NOT NULL,
  description TEXT,
  recommended_action TEXT NOT NULL,

  -- Entity reference (optional link to specific task, phase, category)
  entity_type VARCHAR(50),
    -- values: task, phase, milestone, category
  entity_id UUID,
  entity_name VARCHAR(255),

  -- Impact estimates
  capital_impact DECIMAL(14,2) NOT NULL DEFAULT 0,
  days_impact INTEGER NOT NULL DEFAULT 0,

  -- Status
  status VARCHAR(20) NOT NULL DEFAULT 'active',
    -- values: active, acknowledged, resolved, expired
  acknowledged_at TIMESTAMP WITH TIME ZONE,
  acknowledged_by UUID REFERENCES users(id),
  resolved_at TIMESTAMP WITH TIME ZONE,
  expires_at TIMESTAMP WITH TIME ZONE,

  -- Timestamps
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE,

  CONSTRAINT fk_interventions_builder CHECK (
    EXISTS (SELECT 1 FROM projects WHERE id = project_id AND builder_id = risk_interventions.builder_id)
  )
);

CREATE INDEX idx_risk_interventions_builder_id ON risk_interventions(builder_id);
CREATE INDEX idx_risk_interventions_project_id ON risk_interventions(project_id);
CREATE INDEX idx_risk_interventions_status ON risk_interventions(status);
CREATE INDEX idx_risk_interventions_severity ON risk_interventions(severity);
CREATE INDEX idx_risk_interventions_priority ON risk_interventions(priority_rank);
```

---

## Industry Benchmarks

### Phase Duration Benchmarks

```sql
CREATE TABLE industry_benchmarks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  unit_type VARCHAR(50) NOT NULL,
    -- values: single_family, townhomes, condos, apartments/multifamily
  phase_name VARCHAR(100) NOT NULL,
  min_days INTEGER NOT NULL,
  typical_days INTEGER NOT NULL,
  max_days INTEGER NOT NULL,
  source VARCHAR(100) NOT NULL DEFAULT 'NAHB',
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  CONSTRAINT unique_industry_benchmark UNIQUE(unit_type, phase_name)
);

-- Seed data example
INSERT INTO industry_benchmarks (unit_type, phase_name, min_days, typical_days, max_days) VALUES
('single_family', 'Pre-Development', 30, 45, 90),
('single_family', 'Site Work', 14, 21, 35),
('single_family', 'Foundation', 14, 21, 35),
('single_family', 'Framing', 21, 30, 45),
...
```

### Budget Category Benchmarks

```sql
CREATE TABLE budget_category_benchmarks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  unit_type VARCHAR(50) NOT NULL,
  category_name VARCHAR(100) NOT NULL,
  min_percent DECIMAL(5,2) NOT NULL,
  typical_percent DECIMAL(5,2) NOT NULL,
  max_percent DECIMAL(5,2) NOT NULL,
  source VARCHAR(100) NOT NULL DEFAULT 'RSMeans',
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  CONSTRAINT unique_budget_benchmark UNIQUE(unit_type, category_name)
);

-- Seed data example
INSERT INTO budget_category_benchmarks (unit_type, category_name, min_percent, typical_percent, max_percent) VALUES
('single_family', 'Foundation', 8, 12, 18),
('single_family', 'Framing', 12, 15, 20),
('single_family', 'Electrical', 6, 10, 14),
...
```

---

## Builder-Specific Benchmarks

Learned benchmarks from completed projects.

```sql
CREATE TABLE builder_benchmarks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  benchmark_type VARCHAR(50) NOT NULL,
    -- values: phase_duration, budget_category, cost_per_unit
  unit_type VARCHAR(50),
  category_name VARCHAR(100),
  phase_name VARCHAR(100),

  -- Statistics
  sample_count INTEGER NOT NULL DEFAULT 0,
  avg_value DECIMAL(14,2) NOT NULL,
  std_dev DECIMAL(14,2),
  min_value DECIMAL(14,2),
  max_value DECIMAL(14,2),

  -- Confidence
  confidence DECIMAL(3,2) NOT NULL DEFAULT 0.5,
    -- 0.0 to 1.0, increases with sample_count

  -- Timestamps
  last_calculated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  CONSTRAINT unique_builder_benchmark UNIQUE(builder_id, benchmark_type, unit_type, category_name, phase_name)
);

CREATE INDEX idx_builder_benchmarks_builder_id ON builder_benchmarks(builder_id);
CREATE INDEX idx_builder_benchmarks_type ON builder_benchmarks(benchmark_type);
```

---

## Project Outcomes

Captures actual outcomes for learning when projects complete.

```sql
CREATE TABLE project_outcomes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,

  -- Final metrics
  actual_duration_days INTEGER,
  actual_total_cost DECIMAL(14,2),
  actual_cost_per_unit DECIMAL(14,2),
  final_arv DECIMAL(14,2),

  -- Phase durations
  phase_durations JSONB,
    -- { "Foundation": 25, "Framing": 32, ... }

  -- Category costs (as percentages)
  category_costs JSONB,
    -- { "Foundation": 12.5, "Framing": 16.2, ... }

  -- Timestamps
  completed_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  CONSTRAINT unique_project_outcome UNIQUE(project_id)
);

CREATE INDEX idx_project_outcomes_builder_id ON project_outcomes(builder_id);
CREATE INDEX idx_project_outcomes_project_id ON project_outcomes(project_id);
```

---

## Risk Level Enum

```typescript
type RiskLevel = 'low' | 'medium' | 'high' | 'critical';

// Score thresholds
const RISK_THRESHOLDS = {
  critical: 75,
  high: 50,
  medium: 25,
  low: 0,
};

function getRiskLevel(score: number): RiskLevel {
  if (score >= 75) return 'critical';
  if (score >= 50) return 'high';
  if (score >= 25) return 'medium';
  return 'low';
}
```

## Intervention Types

```typescript
type InterventionType =
  | 'overdue_task'
  | 'blocked_task'
  | 'phase_delay'
  | 'milestone_slip'
  | 'over_budget'
  | 'burn_rate'
  | 'cost_variance';

type InterventionStatus =
  | 'active'
  | 'acknowledged'
  | 'resolved'
  | 'expired';

type InterventionEntityType =
  | 'task'
  | 'phase'
  | 'milestone'
  | 'category';
```

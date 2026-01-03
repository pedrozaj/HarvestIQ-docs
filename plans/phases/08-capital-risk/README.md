# Phase 8: Capital Risk Intelligence

## Overview

Phase 8 transforms HarvestIQ from a project management tool into a proactive capital protection system. The Capital Exposure dashboard becomes the default landing page and command center, answering: **"Where do I need to intervene right now to protect time and capital?"**

## Key Features

### 1. Capital Exposure Dashboard
Portfolio-wide view of risk across all active projects with ranked interventions.

### 2. Risk Calculation Engine
Composite risk scoring combining schedule and budget metrics with trend analysis.

### 3. Required Interventions
Automatically generated action items with severity levels and impact estimates.

### 4. Predictive ARV
AI-powered After Repair Value predictions using market data from FRED API.

### 5. Industry Benchmarks
Compare project metrics against NAHB/RSMeans standards and builder-specific learned baselines.

## UI Philosophy

| Category | Purpose | Examples |
|----------|---------|----------|
| **Primary** | Command center for intervention | Capital Exposure |
| **Evidence** | Data supporting risk analysis | Projects, Schedule, Budget, Tasks |
| **Resources** | Supporting operations | Contractors, Team, Alerts, Jostin |

## Database Tables

| Table | Purpose |
|-------|---------|
| `project_risk_metrics` | Calculated risk scores per project |
| `risk_interventions` | Active and historical interventions |
| `industry_benchmarks` | NAHB/RSMeans phase duration benchmarks |
| `budget_category_benchmarks` | Budget category percentage benchmarks |
| `builder_benchmarks` | Learned benchmarks from completed projects |
| `project_outcomes` | Actual outcomes for learning |

## API Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/risk/dashboard` | Portfolio risk summary |
| GET | `/risk/projects/:id` | Project risk detail |
| POST | `/risk/recalculate` | Recalculate all risks |
| GET | `/risk/interventions` | List interventions |
| POST | `/risk/interventions/:id/acknowledge` | Acknowledge |
| GET | `/risk/benchmarks` | Industry benchmarks |
| POST | `/projects/:id/predict-arv` | Generate ARV prediction |

## Frontend Routes

| Route | Component | Purpose |
|-------|-----------|---------|
| `/capital-risk` | CapitalRiskPage | Portfolio dashboard |
| `/capital-risk/:projectId` | ProjectRiskDetailPage | Project risk detail |

## Implementation Status

- [x] Database schema and migrations
- [x] Risk calculation service
- [x] Intervention generation
- [x] Risk API endpoints
- [x] Capital Exposure dashboard UI
- [x] Project risk detail page
- [x] ARV prediction service
- [x] Market data integration (FRED API)
- [x] Property valuation fields
- [x] Portfolio valuation totals

## Dependencies

- Phase 3 (Schedule & Budget) - Task and budget data for risk calculation
- Phase 4 (Documents) - Appraisal document linking
- Phase 7 (AI Integration) - Claude API for ARV predictions

## External Services

| Service | Purpose |
|---------|---------|
| FRED API | Housing Price Index data for market appreciation |
| Claude API | AI-powered ARV prediction analysis |

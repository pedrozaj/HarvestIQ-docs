# Phase 8: Capital Risk UI

## Capital Exposure Dashboard (`/capital-risk`)

The default landing page and command center for the platform.

### Header Section
- Title: "Capital Exposure"
- Subtitle: "Where to intervene to protect time and capital"
- Recalculate button (triggers risk recalculation for all projects)

### Portfolio Summary Card

```
┌─────────────────────────────────────────────────────────────────┐
│  Total Capital at Risk                                          │
│  $125,000  across 5 projects                                    │
│                                                                 │
│  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐                            │
│  │  1  │  │  2  │  │  1  │  │  1  │                            │
│  │ Red │  │ Org │  │ Yel │  │ Grn │                            │
│  └─────┘  └─────┘  └─────┘  └─────┘                            │
│  Critical  High    Medium   Low                                 │
│                                                                 │
│  Portfolio Exposure Level  ████████░░░░░░░  42                  │
│                                                                 │
│  ──────────────────────────────────────────────────────────    │
│  Unrealized Gain              Forecasted vs Target              │
│  +$200,000 (+6.7%)           +$150,000 (+3.3%)                  │
│  Current: $3.2M vs Target    Predicted: $4.65M vs Target        │
└─────────────────────────────────────────────────────────────────┘
```

### Main Content Grid (2/3 + 1/3 layout)

#### Project Risk Overview Table (Left 2/3)

| Project | Schedule | Budget | Capital at Risk | Trend | Issues |
|---------|----------|--------|-----------------|-------|--------|
| Riverside | ████░░ 70 | ███░░░ 55 | $45,000 | ↗ | 3 overdue, 1 blocked |
| Oak Grove | ███░░░ 45 | ██░░░░ 35 | $25,000 | → | |
| ... | ... | ... | ... | ... | ... |

- Click row to navigate to project risk detail
- Visual progress bars for schedule and budget risk
- Trend indicators: ↗ (worsening), → (stable), ↘ (improving)
- Issue badges highlight key problems

#### Required Interventions Panel (Right 1/3)

```
┌─────────────────────────────────────────┐
│  Required Interventions    10 requiring │
│                                         │
│  ┌───────────────────────────────────┐  │
│  │ 🔴 CRITICAL                       │  │
│  │ Foundation inspection overdue     │  │
│  │ Riverside Homes                   │  │
│  │ 💰 $15,000  📅 5 days            │  │
│  │ [Acknowledge]                     │  │
│  └───────────────────────────────────┘  │
│                                         │
│  ┌───────────────────────────────────┐  │
│  │ 🟠 HIGH                           │  │
│  │ Electrical phase blocked          │  │
│  │ Oak Grove                         │  │
│  │ 💰 $8,000  📅 3 days             │  │
│  │ [Acknowledge]                     │  │
│  └───────────────────────────────────┘  │
│  ...                                    │
└─────────────────────────────────────────┘
```

---

## Project Risk Detail (`/capital-risk/:projectId`)

### Breadcrumb Navigation
`Capital Exposure / Riverside Homes`

### Header
- Project name with risk level badge
- Units and status info
- View Project link
- Recalculate button

### Risk Score Gauges (4-column grid)

```
┌───────────────┐ ┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│   Composite   │ │   Schedule    │ │    Budget     │ │  Capital at   │
│               │ │               │ │               │ │     Risk      │
│     65/100    │ │     70/100    │ │     55/100    │ │               │
│     HIGH      │ │               │ │               │ │    $45,000    │
│   Worsening   │ │               │ │               │ │               │
└───────────────┘ └───────────────┘ └───────────────┘ └───────────────┘
```

### Property Valuation Forecast Card

```
┌─────────────────────────────────────────────────────────────────┐
│  Property Valuation Forecast           [Generate AI Prediction] │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Target ARV  │  │   Current    │  │  Predicted   │          │
│  │   $4.5M      │  │ Appraised    │  │ at Completion│          │
│  │ $450K × 10   │  │   $3.2M      │  │    $4.65M    │          │
│  │ Original goal│  │ $320K × 10   │  │ $465K × 10   │          │
│  │              │  │ -3.3% vs tgt │  │ +3.3% vs tgt │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │ 🟢 Forecast vs Target: +$150,000 (+3.3%)                 │ │
│  │ AI predicts exceeding target ARV at completion           │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

#### AI Prediction Result (when generated)

```
┌─────────────────────────────────────────────────────────────────┐
│  AI Prediction Result                        72% confidence     │
│  $465,000 / unit                             TX market: 5.2%/yr │
│  Total: $4,650,000                                              │
│                                                                 │
│  Based on Texas market appreciation rate of 5.2% annually,     │
│  projected over 8 months remaining. Current appraised value    │
│  is tracking above target, suggesting strong final outcome.    │
│                                                                 │
│  [Apply to Project]  [Dismiss]                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Metrics Cards (2-column grid)

#### Schedule Metrics
| Metric | Value |
|--------|-------|
| Overdue Tasks | 3 |
| Blocked Tasks | 1 |
| Phase Delay | 7 days |
| Milestone Slippage | 3 days |

Schedule Progress: ████████░░░░ 53%
95 days elapsed, 85 days remaining

#### Budget Metrics
| Metric | Value |
|--------|-------|
| Budget Variance | +8.5% |
| Over-Budget Categories | 2 |
| Burn Rate Deviation | +12.3% |
| Cost/Unit vs Benchmark | +5.2% |

### Budget vs Industry Benchmarks Table

| Category | Actual Spend | Actual % | Benchmark % | Variance |
|----------|--------------|----------|-------------|----------|
| Framing | $125,000 | 18.5% | 15.0% | +3.5% |
| Electrical | $85,000 | 12.6% | 10.0% | +2.6% |
| Foundation | $95,000 | 14.1% | 12.0% | +2.1% |
| ... | ... | ... | ... | ... |

- Rows sorted by variance (highest first)
- Color coding: Red (>5%), Orange (>2%), Green (<-2%)

### Project Interventions

Same layout as dashboard interventions panel, filtered to current project.

---

## Components

### RiskLevelBadge
Displays risk level with color coding.

```tsx
<RiskLevelBadge level="high" size="lg" />
```

### RiskScoreGauge
Circular gauge showing score 0-100 with color based on level.

```tsx
<RiskScoreGauge score={65} level="high" label="Composite Risk" size="lg" />
```

### TrendIndicator
Arrow indicator with velocity.

```tsx
<TrendIndicator trend="worsening" velocity={5} />
```

### InterventionCard
Card displaying intervention details with acknowledge button.

```tsx
<InterventionCard intervention={intervention} onAcknowledge={handleAck} />
```

### PortfolioSummaryCard
Full portfolio summary with risk distribution and valuation totals.

```tsx
<PortfolioSummaryCard portfolio={portfolio} />
```

### ProjectRiskTable
Table of all projects with risk metrics.

```tsx
<ProjectRiskTable projects={projectRisks} />
```

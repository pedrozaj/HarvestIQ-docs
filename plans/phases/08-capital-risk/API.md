# Phase 8: Capital Risk API

## Risk Dashboard

### GET /api/risk/dashboard

Returns portfolio-wide risk summary with all project risks and top interventions.

**Response:**
```json
{
  "portfolio": {
    "totalProjects": 5,
    "criticalCount": 1,
    "highCount": 2,
    "mediumCount": 1,
    "lowCount": 1,
    "totalCapitalAtRisk": 125000,
    "avgCompositeScore": 42.5,
    "totalPurchasePrice": 2500000,
    "totalAppraisedValue": 3200000,
    "totalTargetArv": 4500000,
    "totalPredictedArv": 4650000,
    "appraisedValueWithPredictions": 2800000
  },
  "topInterventions": [
    {
      "id": "uuid",
      "projectId": "uuid",
      "projectName": "Riverside Homes",
      "type": "overdue_task",
      "severity": "critical",
      "title": "Foundation inspection overdue",
      "recommendedAction": "Schedule inspection immediately",
      "entityType": "task",
      "entityId": "uuid",
      "entityName": "Foundation Inspection",
      "capitalImpact": 15000,
      "daysImpact": 5,
      "createdAt": "2025-01-15T10:00:00Z"
    }
  ],
  "projectRisks": [
    {
      "project": {
        "id": "uuid",
        "name": "Riverside Homes",
        "unitType": "single_family",
        "units": 10,
        "status": "active",
        "purchasePrice": 500000,
        "appraisedValuePerUnit": 320000,
        "targetArvPerUnit": 450000,
        "predictedArvPerUnit": 465000,
        "appraisedValue": 3200000,
        "targetArv": 4500000,
        "predictedArv": 4650000
      },
      "metrics": {
        "compositeScore": 65,
        "riskLevel": "high",
        "capitalAtRisk": 45000,
        "scheduleScore": 70,
        "budgetScore": 55,
        "trend": "worsening",
        "overdueTasksCount": 3,
        "blockedTasksCount": 1,
        "budgetVariancePercent": 8.5
      },
      "calculatedAt": "2025-01-15T10:00:00Z"
    }
  ]
}
```

---

## Project Risk Detail

### GET /api/risk/projects/:id

Returns detailed risk analysis for a specific project.

**Response:**
```json
{
  "project": {
    "id": "uuid",
    "name": "Riverside Homes",
    "unitType": "single_family",
    "units": 10,
    "status": "active",
    "purchasePrice": 500000,
    "appraisedValue": 320000,
    "targetArv": 450000,
    "predictedArv": 465000
  },
  "metrics": {
    "compositeRiskScore": 65,
    "riskLevel": "high",
    "capitalAtRisk": 45000,
    "scheduleRiskScore": 70,
    "budgetRiskScore": 55,
    "trend": "worsening",
    "trendVelocity": 5,
    "overdueTasksCount": 3,
    "blockedTasksCount": 1,
    "phaseDelayDays": 7,
    "milestoneSlippageDays": 3,
    "budgetVariancePercent": 8.5,
    "overBudgetCategoriesCount": 2,
    "burnRateDeviation": 12.3,
    "costPerUnitVariance": 5.2,
    "calculatedAt": "2025-01-15T10:00:00Z"
  },
  "interventions": [
    {
      "id": "uuid",
      "interventionType": "overdue_task",
      "severity": "critical",
      "priorityRank": 1,
      "title": "Foundation inspection overdue",
      "description": "Task is 5 days past due date",
      "recommendedAction": "Schedule inspection immediately",
      "entityType": "task",
      "entityId": "uuid",
      "entityName": "Foundation Inspection",
      "capitalImpact": 15000,
      "daysImpact": 5,
      "status": "active",
      "createdAt": "2025-01-15T10:00:00Z"
    }
  ],
  "benchmarkComparison": {
    "budgetCategories": [
      {
        "name": "Framing",
        "actual": 125000,
        "actualPercent": 18.5,
        "benchmarkTypical": 15.0,
        "variance": 3.5
      }
    ],
    "scheduleSummary": {
      "totalPlannedDays": 180,
      "elapsed": 95,
      "remaining": 85,
      "progressPercent": 52.8
    }
  }
}
```

---

## Recalculate Risk

### POST /api/risk/recalculate

Recalculates risk metrics for all active projects.

**Response:**
```json
{
  "projectsCalculated": 5,
  "message": "Risk recalculated for 5 projects"
}
```

---

## Interventions

### GET /api/risk/interventions

Returns all active interventions across the portfolio.

**Query Parameters:**
- `status` - Filter by status (active, acknowledged, resolved, expired)
- `severity` - Filter by severity (critical, high, medium, low)
- `type` - Filter by intervention type
- `limit` - Number of results (default 50)
- `offset` - Pagination offset

**Response:**
```json
{
  "interventions": [...],
  "total": 15,
  "limit": 50,
  "offset": 0
}
```

### POST /api/risk/interventions/:id/acknowledge

Acknowledges an intervention.

**Response:**
```json
{
  "id": "uuid",
  "status": "acknowledged",
  "acknowledgedAt": "2025-01-15T10:30:00Z",
  "acknowledgedBy": "uuid"
}
```

---

## Benchmarks

### GET /api/risk/benchmarks

Returns industry benchmarks by unit type.

**Query Parameters:**
- `unitType` - Filter by unit type

**Response:**
```json
{
  "phaseDuration": [
    {
      "unitType": "single_family",
      "phaseName": "Foundation",
      "minDays": 14,
      "typicalDays": 21,
      "maxDays": 35
    }
  ],
  "budgetCategories": [
    {
      "unitType": "single_family",
      "categoryName": "Foundation",
      "minPercent": 8,
      "typicalPercent": 12,
      "maxPercent": 18
    }
  ]
}
```

### POST /api/risk/benchmarks/recalculate

Recalculates builder-specific benchmarks from completed projects.

---

## ARV Prediction

### POST /api/projects/:id/predict-arv

Generates an AI-powered ARV prediction for the project.

**Request Body:**
```json
{
  "save": false
}
```

**Response:**
```json
{
  "predictedArvPerUnit": 465000,
  "predictedArvTotal": 4650000,
  "confidence": "medium",
  "confidencePercent": 72,
  "reasoning": "Based on Texas market appreciation rate of 5.2% annually, projected over 8 months remaining. Current appraised value is tracking above target, suggesting strong final outcome.",
  "factors": {
    "baseProjection": 460000,
    "marketAdjustment": 8000,
    "riskAdjustment": -3000
  },
  "marketData": {
    "region": "TX",
    "appreciationRate": 0.052,
    "source": "FRED"
  },
  "generatedAt": "2025-01-15T10:00:00Z"
}
```

**Errors:**
- `400` - Current appraised value is required
- `404` - Project not found

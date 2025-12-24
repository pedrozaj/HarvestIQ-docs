# Phase 7: AI Processing Pipeline

Query processing, RAG, and insight generation architecture.

---

## Query Processing Flow

```
User Question
    ↓
Intent Classification (LLM)
    ↓
Entity Extraction (LLM)
    ↓
Query Strategy Selection
    ↓
Database Query / Vector Search / LLM Analysis
    ↓
Response Generation (LLM)
    ↓
Store in conversation history
    ↓
Return to user
```

---

## Intent Classification

First LLM call classifies the query type.

```typescript
// src/services/ai/classify.ts
const INTENT_PROMPT = `
Classify this user question into one of these categories:
- spending: Questions about costs, payments, expenses
- schedule: Questions about tasks, timelines, bottlenecks
- budget: Questions about budget vs actual, variance
- comparison: Questions comparing projects or categories
- trend: Questions about patterns over time
- document: Questions about document content
- open_ended: General questions, recommendations

Question: "{question}"

Respond with just the category name.
`;

async function classifyIntent(question: string): Promise<QueryType> {
  const response = await claude.complete(INTENT_PROMPT.replace('{question}', question));
  return response.trim() as QueryType;
}
```

---

## Entity Extraction

Second LLM call extracts relevant entities.

```typescript
// src/services/ai/extract.ts
const EXTRACTION_PROMPT = `
Extract entities from this question about construction projects.

Question: "{question}"
Context: Builder has projects: {projectNames}
Categories: {categoryNames}

Extract these if mentioned:
- project_name: specific project mentioned
- category_name: budget category mentioned
- time_range: date range mentioned (e.g., "last month", "this year")
- comparison_targets: items being compared

Respond in JSON format.
`;

interface ExtractedEntities {
  projectId?: string;
  categoryId?: string;
  timeRange?: { start: string; end: string };
  comparisonTargets?: string[];
}
```

---

## Query Strategy

Select data retrieval method based on intent and entities.

```typescript
// src/services/ai/strategy.ts
interface QueryStrategy {
  type: 'database' | 'vector_search' | 'hybrid';
  queries: string[];
  ragRequired: boolean;
}

function selectStrategy(intent: QueryType, entities: ExtractedEntities): QueryStrategy {
  switch (intent) {
    case 'spending':
    case 'budget':
      return {
        type: 'database',
        queries: ['payment_summary', 'budget_variance'],
        ragRequired: false,
      };

    case 'schedule':
      return {
        type: 'database',
        queries: ['task_status', 'blocked_tasks', 'overdue_tasks'],
        ragRequired: false,
      };

    case 'document':
      return {
        type: 'vector_search',
        queries: ['document_embeddings'],
        ragRequired: true,
      };

    case 'open_ended':
      return {
        type: 'hybrid',
        queries: ['project_summary', 'recent_activity'],
        ragRequired: true,
      };

    default:
      return { type: 'database', queries: ['general'], ragRequired: false };
  }
}
```

---

## Database Queries

Pre-defined queries for each intent type.

```typescript
// src/services/ai/queries.ts
const QUERY_MAP = {
  payment_summary: (builderId: string, filters: object) => `
    SELECT bc.name as category, SUM(p.amount) as total
    FROM payments p
    JOIN budget_categories bc ON p.category_id = bc.id
    WHERE p.builder_id = $1
    ${filters.projectId ? 'AND p.project_id = $2' : ''}
    GROUP BY bc.id
    ORDER BY total DESC
  `,

  blocked_tasks: (builderId: string, projectId?: string) => `
    SELECT t.*, p.name as project_name,
           array_agg(dt.name) as blocking_tasks
    FROM schedule_tasks t
    JOIN projects p ON t.project_id = p.id
    JOIN task_dependencies td ON td.task_id = t.id
    JOIN schedule_tasks dt ON td.depends_on_task_id = dt.id AND dt.status != 'completed'
    WHERE t.builder_id = $1 AND t.status = 'blocked'
    GROUP BY t.id, p.name
  `,

  // ... more query templates
};
```

---

## Vector Search (RAG)

For document content queries.

```typescript
// src/services/ai/rag.ts
async function vectorSearch(
  builderId: string,
  question: string,
  limit: number = 5
): Promise<DocumentChunk[]> {
  // Generate embedding for question
  const questionEmbedding = await openai.embeddings.create({
    model: 'text-embedding-ada-002',
    input: question,
  });

  // Search for similar chunks
  const results = await db.query(`
    SELECT
      de.chunk_text,
      de.chunk_index,
      d.id as document_id,
      d.name as document_name,
      d.type as document_type,
      1 - (de.embedding <=> $1) as similarity
    FROM document_embeddings de
    JOIN documents d ON de.document_id = d.id
    WHERE d.builder_id = $2
    ORDER BY de.embedding <=> $1
    LIMIT $3
  `, [questionEmbedding.data[0].embedding, builderId, limit]);

  return results.rows;
}
```

---

## Response Generation

Final LLM call generates user response.

```typescript
// src/services/ai/generate.ts
const RESPONSE_PROMPT = `
You are an AI assistant for a construction project management app.

User Question: {question}
Context: {context}
Data Retrieved: {data}

Generate a helpful, concise response that:
1. Directly answers the question
2. Uses specific numbers and facts from the data
3. Highlights any concerns or insights
4. Is conversational but professional

If the data doesn't fully answer the question, acknowledge what you can answer.
`;

async function generateResponse(
  question: string,
  context: object,
  data: object[]
): Promise<string> {
  const prompt = RESPONSE_PROMPT
    .replace('{question}', question)
    .replace('{context}', JSON.stringify(context))
    .replace('{data}', JSON.stringify(data));

  const response = await claude.messages.create({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 1024,
    messages: [{ role: 'user', content: prompt }],
  });

  return response.content[0].text;
}
```

---

## Document Embedding Pipeline

Process uploaded documents for RAG.

```
Document Uploaded
    ↓
Queue text extraction job
    ↓
Extract text (PDF parsing, OCR if needed)
    ↓
Store extracted_text in documents table
    ↓
Chunk text into segments (500-1000 tokens)
    ↓
Generate embedding for each chunk
    ↓
Store in document_embeddings table
```

```typescript
// src/jobs/document-embedding.job.ts
async function processDocumentEmbeddings(job: Job) {
  const { documentId, builderId } = job.data;

  // Get extracted text
  const doc = await db.query(
    'SELECT extracted_text FROM documents WHERE id = $1',
    [documentId]
  );

  if (!doc.rows[0]?.extracted_text) return;

  // Chunk text
  const chunks = chunkText(doc.rows[0].extracted_text, {
    maxTokens: 500,
    overlap: 50,
  });

  // Generate and store embeddings
  for (let i = 0; i < chunks.length; i++) {
    const embedding = await openai.embeddings.create({
      model: 'text-embedding-ada-002',
      input: chunks[i],
    });

    await db.query(`
      INSERT INTO document_embeddings (builder_id, document_id, chunk_index, chunk_text, embedding)
      VALUES ($1, $2, $3, $4, $5)
    `, [builderId, documentId, i, chunks[i], embedding.data[0].embedding]);
  }
}
```

---

## Insight Generation

Background job analyzes data and generates insights.

```typescript
// src/jobs/insight-generation.job.ts
async function generateInsights(job: Job) {
  const { builderId } = job.data;

  // Check budget anomalies
  const budgetAnomalies = await detectBudgetAnomalies(builderId);
  for (const anomaly of budgetAnomalies) {
    await createInsight({
      builderId,
      projectId: anomaly.projectId,
      type: 'cost_pattern',
      severity: anomaly.percentOver > 20 ? 'warning' : 'info',
      title: `${anomaly.categoryName} costs above average`,
      description: `${anomaly.projectName} has spent ${anomaly.percentOver}% more than your typical ${anomaly.categoryName} costs.`,
      data: anomaly,
    });
  }

  // Check schedule risks
  const scheduleRisks = await detectScheduleRisks(builderId);
  // ... create insights

  // Check efficiency patterns
  const efficiencyPatterns = await analyzeEfficiency(builderId);
  // ... create insights
}
```

---

## Token Usage Tracking

Track and limit token usage per builder.

```typescript
// src/services/ai/tokens.ts
async function checkTokenLimit(builderId: string): Promise<boolean> {
  const builder = await db.query(
    'SELECT ai_tokens_used_month, ai_tokens_limit FROM builders WHERE id = $1',
    [builderId]
  );

  return builder.rows[0].ai_tokens_used_month < builder.rows[0].ai_tokens_limit;
}

async function recordTokenUsage(builderId: string, tokens: number): Promise<void> {
  await db.query(
    'UPDATE builders SET ai_tokens_used_month = ai_tokens_used_month + $1 WHERE id = $2',
    [tokens, builderId]
  );
}

// Monthly reset job
async function resetMonthlyTokens() {
  await db.query(`
    UPDATE builders
    SET ai_tokens_used_month = 0,
        ai_tokens_reset_at = DATE_TRUNC('month', NOW()) + INTERVAL '1 month'
    WHERE ai_tokens_reset_at <= NOW()
  `);
}
```

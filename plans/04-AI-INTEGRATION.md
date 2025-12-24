# Phase 4: AI Integration

Conversational interface layer for querying and insights.

---

## Natural Language Query System

### Interface
- Chat-style input available from any page
- Floating/collapsible chat panel
- Context-aware (knows current project if on project page)
- Conversation history within session and persisted

### Query Processing Pipeline
```
User Question
    ↓
Intent Classification (LLM)
    ↓
Entity Extraction (LLM)
    ↓
Query Strategy Selection
    ↓
Database Query / LLM Analysis
    ↓
Response Generation (LLM)
    ↓
Formatted Answer to User
    ↓
Store in conversation history
```

---

## AI Data Structures

### Conversations
```sql
CREATE TABLE ai_conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  user_id UUID NOT NULL REFERENCES users(id),
  project_id UUID REFERENCES projects(id),
  title VARCHAR(255),
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE,

  -- Ensure user belongs to same builder
  CONSTRAINT fk_ai_conversations_user_same_builder
    CHECK (EXISTS (
      SELECT 1 FROM users WHERE id = user_id AND builder_id = ai_conversations.builder_id
    )),
  -- Ensure project belongs to same builder
  CONSTRAINT fk_ai_conversations_project_same_builder
    CHECK (project_id IS NULL OR EXISTS (
      SELECT 1 FROM projects WHERE id = project_id AND builder_id = ai_conversations.builder_id
    ))
);

CREATE INDEX idx_ai_conversations_builder_id ON ai_conversations(builder_id);
CREATE INDEX idx_ai_conversations_user_id ON ai_conversations(user_id);
CREATE INDEX idx_ai_conversations_project_id ON ai_conversations(project_id);
CREATE INDEX idx_ai_conversations_created_at ON ai_conversations(created_at);
```

### Conversation Messages
```sql
CREATE TABLE ai_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  conversation_id UUID NOT NULL REFERENCES ai_conversations(id) ON DELETE CASCADE,
  role VARCHAR(20) NOT NULL,
    -- values: user, assistant
  content TEXT NOT NULL,
  query_type VARCHAR(50),
    -- values: spending, schedule, budget, comparison, trend, document, open_ended
  query_params JSONB,
  sources JSONB,
  tokens_used INTEGER,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  -- Ensure conversation belongs to same builder
  CONSTRAINT fk_ai_messages_conversation_same_builder
    CHECK (EXISTS (
      SELECT 1 FROM ai_conversations WHERE id = conversation_id AND builder_id = ai_messages.builder_id
    ))
);

CREATE INDEX idx_ai_messages_builder_id ON ai_messages(builder_id);
CREATE INDEX idx_ai_messages_conversation_id ON ai_messages(conversation_id);
CREATE INDEX idx_ai_messages_created_at ON ai_messages(created_at);
```

### Document Embeddings
```sql
CREATE TABLE document_embeddings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  document_id UUID NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  chunk_index INTEGER NOT NULL,
  chunk_text TEXT NOT NULL,
  embedding VECTOR(1536),
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  -- Ensure document belongs to same builder
  CONSTRAINT fk_document_embeddings_doc_same_builder
    CHECK (EXISTS (
      SELECT 1 FROM documents WHERE id = document_id AND builder_id = document_embeddings.builder_id
    ))
);

CREATE INDEX idx_document_embeddings_builder_id ON document_embeddings(builder_id);
CREATE INDEX idx_document_embeddings_document_id ON document_embeddings(document_id);
CREATE INDEX idx_document_embeddings_embedding ON document_embeddings
  USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

### AI Insights
```sql
CREATE TABLE ai_insights (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  project_id UUID REFERENCES projects(id),
  insight_type VARCHAR(50) NOT NULL,
    -- values: cost_pattern, schedule_risk, efficiency, anomaly, recommendation
  title VARCHAR(255) NOT NULL,
  description TEXT NOT NULL,
  data JSONB,
  severity VARCHAR(20) NOT NULL DEFAULT 'info',
    -- values: info, warning, critical
  is_dismissed BOOLEAN NOT NULL DEFAULT false,
  dismissed_by UUID REFERENCES users(id),
  dismissed_at TIMESTAMP WITH TIME ZONE,
  expires_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  -- Ensure project belongs to same builder
  CONSTRAINT fk_ai_insights_project_same_builder
    CHECK (project_id IS NULL OR EXISTS (
      SELECT 1 FROM projects WHERE id = project_id AND builder_id = ai_insights.builder_id
    )),
  -- Ensure dismissed_by user belongs to same builder
  CONSTRAINT fk_ai_insights_dismisser_same_builder
    CHECK (dismissed_by IS NULL OR EXISTS (
      SELECT 1 FROM users WHERE id = dismissed_by AND builder_id = ai_insights.builder_id
    ))
);

CREATE INDEX idx_ai_insights_builder_id ON ai_insights(builder_id);
CREATE INDEX idx_ai_insights_project_id ON ai_insights(project_id);
CREATE INDEX idx_ai_insights_insight_type ON ai_insights(insight_type);
CREATE INDEX idx_ai_insights_is_dismissed ON ai_insights(is_dismissed);
CREATE INDEX idx_ai_insights_created_at ON ai_insights(created_at);
```

### AI Query Feedback
```sql
CREATE TABLE ai_feedback (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  message_id UUID NOT NULL REFERENCES ai_messages(id),
  user_id UUID NOT NULL REFERENCES users(id),
  rating INTEGER,
    -- values: 1-5 or thumbs up/down (1 or -1)
  feedback_text TEXT,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  -- Ensure message belongs to same builder
  CONSTRAINT fk_ai_feedback_message_same_builder
    CHECK (EXISTS (
      SELECT 1 FROM ai_messages WHERE id = message_id AND builder_id = ai_feedback.builder_id
    )),
  -- Ensure user belongs to same builder
  CONSTRAINT fk_ai_feedback_user_same_builder
    CHECK (EXISTS (
      SELECT 1 FROM users WHERE id = user_id AND builder_id = ai_feedback.builder_id
    ))
);

CREATE INDEX idx_ai_feedback_builder_id ON ai_feedback(builder_id);
CREATE INDEX idx_ai_feedback_message_id ON ai_feedback(message_id);
```

### Learned Patterns (Baselines)
```sql
CREATE TABLE ai_patterns (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  pattern_type VARCHAR(50) NOT NULL,
    -- values: category_cost_baseline, phase_duration_baseline, cost_per_unit_baseline
  category_id UUID REFERENCES budget_categories(id),
  pattern_data JSONB NOT NULL,
    -- e.g., { "avg_cost": 15000, "std_dev": 2500, "sample_size": 5 }
  confidence DECIMAL(3,2),
  last_calculated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

  -- Ensure category belongs to same builder
  CONSTRAINT fk_ai_patterns_category_same_builder
    CHECK (category_id IS NULL OR EXISTS (
      SELECT 1 FROM budget_categories WHERE id = category_id AND builder_id = ai_patterns.builder_id
    ))
);

CREATE INDEX idx_ai_patterns_builder_id ON ai_patterns(builder_id);
CREATE INDEX idx_ai_patterns_pattern_type ON ai_patterns(pattern_type);
CREATE INDEX idx_ai_patterns_category_id ON ai_patterns(category_id);
```

---

## Query Processing Approaches

### Standard Database Queries
For structured, quantifiable questions with clear parameters:
- Sum, count, average calculations
- Filtering by known fields (date, category, project)
- Status lookups
- List retrievals

### LLM-Powered Analysis
For questions requiring interpretation, inference, or unstructured analysis:
- Pattern recognition across data
- Trend analysis and explanations
- Bottleneck identification (requires understanding dependencies)
- Recommendations and suggestions
- Comparison analysis with context
- Document content search and summarization
- Ambiguous or open-ended questions

### Hybrid Approach
Most queries combine both:
1. LLM interprets the question and extracts parameters
2. Database queries retrieve relevant data
3. LLM synthesizes the data into a human response

---

## Query Categories

The following are representative examples. The system handles variations and evolving query patterns through LLM interpretation.

### Spending Queries
- "How much did I spend on plumbing?"
- "What's my total spend on Project X?"
- "What's my cost per unit on Riverside?"
- "What's my largest expense this month?"

**Approach:** Primarily database aggregation, LLM for parameter extraction and response formatting.

### Schedule Queries
- "Where are my bottlenecks?"
- "What tasks are overdue?"
- "When is the next milestone?"
- "What's blocking progress on Project X?"

**Approach:** Database queries for status/date filtering, LLM for identifying bottlenecks and explaining dependency chains.

### Budget Queries
- "Am I over budget on electrical?"
- "How much budget is remaining?"
- "Which categories are over budget?"
- "What's my estimated vs actual for framing?"

**Approach:** Database aggregation, LLM for contextual explanation of variances.

### Comparison Queries
- "Which project is most efficient?"
- "How does Project X compare to Project Y on plumbing costs?"
- "What's my average cost per project for permits?"

**Approach:** Database aggregation for metrics, LLM for meaningful comparison narrative.

### Trend Analysis
- "Where am I spending the most money?"
- "Are my projects getting more efficient?"
- "What's my spending trend over the last 6 months?"

**Approach:** Database for time-series data, LLM for pattern identification and explanation.

### Document Queries
- "Show me all invoices for plumbing"
- "What estimates do I have for Project X?"
- "Find the contract for [subcontractor]"
- "What does invoice #1234 say about payment terms?"

**Approach:** Database for metadata filtering, vector search for content queries, LLM for summarization.

### Open-Ended Questions
- "What should I focus on today?"
- "Are there any concerns with Project X?"
- "How can I reduce costs on my next project?"

**Approach:** Primarily LLM with database context, requires understanding of user's role and project state.

---

## RAG (Retrieval Augmented Generation) for Documents

### Document Processing Pipeline
```
Document Uploaded
    ↓
Text Extraction Job Queued
    ↓
Extract text (PDF parsing, OCR if needed)
    ↓
Store extracted_text in documents table
    ↓
Chunk text into segments (500-1000 tokens each)
    ↓
Generate embeddings for each chunk
    ↓
Store in document_embeddings table
```

### Document Query Flow
```
User asks document-related question
    ↓
Generate embedding for question
    ↓
Vector similarity search in document_embeddings
    ↓
Retrieve top-k relevant chunks
    ↓
Include chunks as context for LLM
    ↓
LLM generates answer with source citations
```

---

## Proactive Insights

### Insight Generation
Background job runs periodically to analyze data and generate insights:
- Compare current project metrics against learned patterns
- Identify anomalies (costs significantly above/below baseline)
- Detect schedule risks (blocked tasks, approaching deadlines)
- Generate efficiency recommendations

### Insight Types
- **Cost Pattern**: "Plumbing costs on Project X are 30% above your average"
- **Schedule Risk**: "3 tasks are blocked, delaying the electrical phase"
- **Efficiency**: "Project Y completed framing 2 weeks faster than average"
- **Anomaly**: "Unusual payment of $15,000 for electrical detected"
- **Recommendation**: "Based on past projects, budget an extra 10% for permits"

### Insight Delivery
- Displayed in dashboard insights section
- Can be dismissed by users
- Optionally sent via notification

---

## API Endpoints (AI)

```
POST /ai/conversations                          - Start new conversation
GET  /ai/conversations                          - List conversations (paginated)
  ?limit=20&offset=0
GET  /ai/conversations/:id                      - Get conversation with messages
DELETE /ai/conversations/:id                    - Delete conversation

POST /ai/conversations/:id/messages             - Send message, get response
  body: { message: string, context?: { projectId?: string } }
  response: { answer: string, sources?: array, queryType?: string }

POST /ai/feedback                               - Submit feedback on response
  body: { messageId: string, rating: number, feedbackText?: string }

GET  /ai/insights                               - Get active insights
  ?project_id=X&insight_type=X&is_dismissed=false&limit=20
POST /ai/insights/:id/dismiss                   - Dismiss insight

GET  /ai/suggestions                            - Get contextual suggestions
  ?project_id=X
  response: { suggestions: string[] }
```

---

## Technical Implementation

### LLM Integration
- Claude API for query understanding and response generation
- Structured output parsing for query parameters
- Context window management for conversation history
- Token usage tracking per message

### Vector Database
- PostgreSQL with pgvector extension
- Install via migration: `CREATE EXTENSION IF NOT EXISTS vector;`
- Embeddings generated via embedding API (OpenAI or similar)
- Approximate nearest neighbor search for performance

### Cost Controls
- AI token usage tracked per builder in `builders.ai_tokens_used_month`
- Monthly limit enforced (default 100k tokens)
- Reset monthly via `ai_tokens_reset_at`
- Reject queries when limit exceeded

### Background Processing
- Document text extraction queued on upload
- Embedding generation after text extraction
- Pattern recalculation on schedule (daily/weekly)
- Insight generation on schedule

---

## Example Interaction Flow

**User:** "How much did I spend on plumbing?"

**Processing:**
1. LLM classifies intent: spending_query
2. LLM extracts entities: category=plumbing, project=all, time=all
3. System executes: `SELECT project_id, SUM(amount) FROM payments JOIN budget_items... WHERE category='plumbing' GROUP BY project_id`
4. LLM formats response with retrieved data
5. Store message and response in ai_messages

**Response:** "You've spent $45,230 on plumbing across all projects. Project A: $18,500, Project B: $15,730, Project C: $11,000."

---

**User:** "Where are my bottlenecks?"

**Processing:**
1. LLM classifies intent: schedule_analysis (bottleneck)
2. System retrieves: blocked tasks, overdue tasks, dependency chains
3. LLM analyzes: identifies root causes, explains impact
4. Store message and response

**Response:** "I found 3 bottlenecks:
1. Project A - Electrical rough-in is blocked waiting for framing inspection (5 days overdue)
2. Project B - Cabinet installation depends on countertop delivery (delayed 2 weeks)
3. Project C - Final inspection waiting on permit approval

The framing inspection in Project A is the most critical - it's blocking 4 downstream tasks."

---

**User:** "What does invoice #4521 say about the warranty?"

**Processing:**
1. LLM classifies intent: document_query
2. System finds invoice by number, retrieves document embeddings
3. Vector search for "warranty" related chunks
4. LLM extracts warranty information from relevant chunks
5. Store message and response with sources

**Response:** "Invoice #4521 from ABC Plumbing includes a 2-year warranty on all labor and a 5-year warranty on parts. The warranty excludes damage from improper use or maintenance."

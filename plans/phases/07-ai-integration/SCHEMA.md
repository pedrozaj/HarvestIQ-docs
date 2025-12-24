# Phase 7: Database Schema

AI conversation, embedding, and insight tables.

---

## Prerequisites

Enable pgvector extension:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

---

## Migration Order

```
022_enable_pgvector.sql
023_create_ai_conversations.sql
024_create_ai_messages.sql
025_create_ai_feedback.sql
026_create_ai_insights.sql
027_create_ai_patterns.sql
028_create_document_embeddings.sql
```

---

## Table: ai_conversations

Chat session records.

```sql
CREATE TABLE ai_conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  user_id UUID NOT NULL REFERENCES users(id),
  project_id UUID REFERENCES projects(id),
  title VARCHAR(255),
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_ai_conversations_builder_id ON ai_conversations(builder_id);
CREATE INDEX idx_ai_conversations_user_id ON ai_conversations(user_id);
CREATE INDEX idx_ai_conversations_project_id ON ai_conversations(project_id);
```

---

## Table: ai_messages

Individual messages in conversations.

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
    -- Extracted parameters
  sources JSONB,
    -- Referenced data sources
  tokens_used INTEGER,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_ai_messages_conversation_id ON ai_messages(conversation_id);
CREATE INDEX idx_ai_messages_created_at ON ai_messages(created_at);
```

---

## Table: ai_feedback

User feedback on AI responses.

```sql
CREATE TABLE ai_feedback (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  message_id UUID NOT NULL REFERENCES ai_messages(id),
  user_id UUID NOT NULL REFERENCES users(id),
  rating INTEGER,
    -- 1 (thumbs down) or 5 (thumbs up)
  feedback_text TEXT,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_ai_feedback_message_id ON ai_feedback(message_id);
```

---

## Table: ai_insights

Proactively generated insights.

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
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_ai_insights_builder_id ON ai_insights(builder_id);
CREATE INDEX idx_ai_insights_project_id ON ai_insights(project_id);
CREATE INDEX idx_ai_insights_is_dismissed ON ai_insights(is_dismissed);
```

---

## Table: ai_patterns

Learned baselines for anomaly detection.

```sql
CREATE TABLE ai_patterns (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  pattern_type VARCHAR(50) NOT NULL,
    -- values: category_cost_baseline, phase_duration_baseline, cost_per_unit_baseline
  category_id UUID REFERENCES budget_categories(id),
  pattern_data JSONB NOT NULL,
    -- { avg_cost: 15000, std_dev: 2500, sample_size: 5 }
  confidence DECIMAL(3,2),
  last_calculated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_ai_patterns_builder_id ON ai_patterns(builder_id);
CREATE INDEX idx_ai_patterns_pattern_type ON ai_patterns(pattern_type);
```

---

## Table: document_embeddings

Vector embeddings for document search (RAG).

```sql
CREATE TABLE document_embeddings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  builder_id UUID NOT NULL REFERENCES builders(id),
  document_id UUID NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  chunk_index INTEGER NOT NULL,
  chunk_text TEXT NOT NULL,
  embedding VECTOR(1536),
    -- OpenAI ada-002 produces 1536 dimensions
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_document_embeddings_document_id ON document_embeddings(document_id);
CREATE INDEX idx_document_embeddings_embedding ON document_embeddings
  USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

---

## Token Usage Tracking

Tracked in `builders` table:

```sql
-- Already exists from Phase 1
ai_tokens_used_month INTEGER NOT NULL DEFAULT 0,
ai_tokens_limit INTEGER NOT NULL DEFAULT 100000,
ai_tokens_reset_at TIMESTAMP WITH TIME ZONE
```

Update after each AI message:

```sql
UPDATE builders
SET ai_tokens_used_month = ai_tokens_used_month + $1
WHERE id = $2;
```

Reset monthly via scheduled job:

```sql
UPDATE builders
SET ai_tokens_used_month = 0,
    ai_tokens_reset_at = DATE_TRUNC('month', NOW()) + INTERVAL '1 month'
WHERE ai_tokens_reset_at <= NOW();
```

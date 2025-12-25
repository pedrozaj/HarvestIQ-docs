# Phase 7: Implementation Checklist

Step-by-step implementation tasks for AI integration.

---

## Prerequisites

- [ ] Phase 6 complete (all data available for querying)
- [ ] Claude API key obtained and configured
- [ ] OpenAI API key obtained (for embeddings)
- [ ] pgvector extension available on Railway PostgreSQL

---

## Database Setup

### Enable pgvector
- [ ] Create migration `026_enable_pgvector.sql`
- [ ] Add `CREATE EXTENSION IF NOT EXISTS vector;`
- [ ] Verify extension enabled in production

### Create Tables
- [ ] Create migration `027_create_ai_conversations.sql`
- [ ] Create migration `028_create_ai_messages.sql`
- [ ] Create migration `029_create_ai_feedback.sql`
- [ ] Create migration `030_create_ai_insights.sql`
- [ ] Create migration `031_create_ai_patterns.sql`
- [ ] Create migration `032_create_document_embeddings.sql`

### Create Indexes
- [ ] Add builder_id indexes on all AI tables
- [ ] Add conversation_id index on ai_messages
- [ ] Add IVFFlat index on document_embeddings.embedding
- [ ] Verify index performance with EXPLAIN ANALYZE

---

## Backend: AI Services

### Token Management
- [ ] Create `src/services/ai/tokens.ts`
- [ ] Implement `checkTokenLimit()` function
- [ ] Implement `recordTokenUsage()` function
- [ ] Add token limit middleware to AI routes
- [ ] Create monthly reset job handler

### Intent Classification
- [ ] Create `src/services/ai/classify.ts`
- [ ] Define intent categories (spending, schedule, budget, comparison, trend, document, open_ended)
- [ ] Create classification prompt template
- [ ] Implement `classifyIntent()` function
- [ ] Add unit tests for classification

### Entity Extraction
- [ ] Create `src/services/ai/extract.ts`
- [ ] Define entity types (projectId, categoryId, timeRange, comparisonTargets)
- [ ] Create extraction prompt template
- [ ] Implement `extractEntities()` function
- [ ] Add fuzzy matching for project/category names

### Query Strategy
- [ ] Create `src/services/ai/strategy.ts`
- [ ] Define strategy types (database, vector_search, hybrid)
- [ ] Implement `selectStrategy()` function
- [ ] Map intents to query strategies

### Database Queries
- [ ] Create `src/services/ai/queries.ts`
- [ ] Implement `payment_summary` query
- [ ] Implement `budget_variance` query
- [ ] Implement `task_status` query
- [ ] Implement `blocked_tasks` query
- [ ] Implement `overdue_tasks` query
- [ ] Implement `project_summary` query
- [ ] Add query result formatting

### Vector Search (RAG)
- [ ] Create `src/services/ai/rag.ts`
- [ ] Implement question embedding generation
- [ ] Implement similarity search query
- [ ] Add relevance score filtering
- [ ] Format chunk results with metadata

### Response Generation
- [ ] Create `src/services/ai/generate.ts`
- [ ] Create response prompt template
- [ ] Implement `generateResponse()` function
- [ ] Add source citation formatting
- [ ] Handle partial/no data scenarios

### Pipeline Orchestration
- [ ] Create `src/services/ai/pipeline.ts`
- [ ] Implement full query pipeline
- [ ] Add error handling at each stage
- [ ] Add timing/metrics collection
- [ ] Create pipeline tests

---

## Backend: API Endpoints

### Conversations
- [ ] Create `src/routes/ai.routes.ts`
- [ ] Implement `GET /ai/conversations` - list conversations
- [ ] Implement `POST /ai/conversations` - create conversation
- [ ] Implement `GET /ai/conversations/:id` - get with messages
- [ ] Implement `DELETE /ai/conversations/:id` - soft delete
- [ ] Add pagination to list endpoint

### Messages
- [ ] Implement `POST /ai/conversations/:id/messages`
- [ ] Store user message
- [ ] Run AI pipeline
- [ ] Store assistant response
- [ ] Return response with sources
- [ ] Handle token limit errors (429)

### Feedback
- [ ] Implement `POST /ai/feedback`
- [ ] Validate message ownership
- [ ] Store rating and optional text
- [ ] Return confirmation

### Insights
- [ ] Implement `GET /ai/insights`
- [ ] Add project filter
- [ ] Add type filter
- [ ] Add dismissed filter
- [ ] Implement `POST /ai/insights/:id/dismiss`
- [ ] Update dismissed_by and dismissed_at

### Suggestions
- [ ] Implement `GET /ai/suggestions`
- [ ] Generate context-aware suggestions
- [ ] Add project context support

### Validation
- [ ] Create Zod schemas for all AI endpoints
- [ ] Add request validation middleware
- [ ] Add response type definitions

---

## Backend: Background Jobs

### Document Embedding Job
- [ ] Create `src/jobs/document-embedding.job.ts`
- [ ] Implement text chunking (500 tokens, 50 overlap)
- [ ] Implement embedding generation
- [ ] Implement batch embedding storage
- [ ] Add job queue trigger on document upload
- [ ] Handle large documents gracefully

### Insight Generation Job
- [ ] Create `src/jobs/insight-generation.job.ts`
- [ ] Implement budget anomaly detection
- [ ] Implement schedule risk detection
- [ ] Implement efficiency pattern analysis
- [ ] Create insight records with proper severity
- [ ] Schedule daily/weekly runs

### Pattern Learning Job
- [ ] Create `src/jobs/pattern-learning.job.ts`
- [ ] Calculate category cost baselines
- [ ] Calculate phase duration baselines
- [ ] Store patterns with confidence scores
- [ ] Update patterns incrementally

### Token Reset Job
- [ ] Create `src/jobs/token-reset.job.ts`
- [ ] Reset monthly token counts
- [ ] Update reset_at timestamp
- [ ] Schedule for first of month

---

## Frontend: Components

### Conversation Components
- [ ] Create `ConversationList` component
- [ ] Create `ConversationItem` component
- [ ] Add new chat button
- [ ] Add date grouping (Today, Yesterday, Last Week)
- [ ] Add active conversation highlighting

### Chat Components
- [ ] Create `ChatArea` component with scroll
- [ ] Create `ChatMessage` component
- [ ] Style user vs assistant messages
- [ ] Add message timestamps
- [ ] Add typing indicator

### Message Components
- [ ] Create `MessageInput` component
- [ ] Add submit on Enter
- [ ] Add character limit display
- [ ] Add loading state during AI response
- [ ] Create `SourcesList` collapsible component
- [ ] Create `FeedbackButtons` component

### Context Components
- [ ] Create `ContextSelector` dropdown
- [ ] Populate with user's projects
- [ ] Add "All Projects" option
- [ ] Persist selection in conversation

### Suggestion Components
- [ ] Create `SuggestionChips` component
- [ ] Fetch suggestions from API
- [ ] Handle chip click to send message

### Warning Components
- [ ] Create `TokenWarning` banner
- [ ] Show at 80% usage
- [ ] Show limit reached state
- [ ] Add upgrade CTA

---

## Frontend: Pages

### AI Chat Page
- [ ] Create `/ai` route
- [ ] Implement three-column layout
- [ ] Add conversation list sidebar
- [ ] Add main chat area
- [ ] Add context selector in header
- [ ] Connect to conversations API
- [ ] Connect to messages API

### Floating AI Panel
- [ ] Create `AIPanel` component
- [ ] Add to app header
- [ ] Implement minimize/expand
- [ ] Inherit context from current page
- [ ] Persist conversation during session
- [ ] Add "Open Full Chat" link

---

## Frontend: State Management

### AI Context
- [ ] Create `AIContext` provider
- [ ] Store current conversation
- [ ] Store panel open state
- [ ] Store context (selected project)

### API Hooks
- [ ] Create `useConversations` hook
- [ ] Create `useConversation` hook
- [ ] Create `useSendMessage` mutation
- [ ] Create `useInsights` hook
- [ ] Create `useSuggestions` hook

---

## Integration

### Document Upload Integration
- [ ] Trigger embedding job after text extraction
- [ ] Show embedding status in document list
- [ ] Handle embedding failures gracefully

### Dashboard Integration
- [ ] Display active insights on dashboard
- [ ] Add dismiss functionality
- [ ] Link insights to relevant pages

### User Profile Integration
- [ ] Display token usage in profile
- [ ] Show reset date
- [ ] Show upgrade options

---

## Testing

### Unit Tests
- [ ] Test intent classification with sample questions
- [ ] Test entity extraction accuracy
- [ ] Test query generation
- [ ] Test response formatting

### Integration Tests
- [ ] Test full query pipeline
- [ ] Test conversation persistence
- [ ] Test token limit enforcement
- [ ] Test RAG search accuracy

### E2E Tests
- [ ] Test asking spending question
- [ ] Test asking schedule question
- [ ] Test document search
- [ ] Test conversation history
- [ ] Test feedback submission

---

## Performance

- [ ] Add caching for repeated queries
- [ ] Optimize embedding search with proper index tuning
- [ ] Add query result caching
- [ ] Monitor API latency
- [ ] Set up Claude API rate limiting

---

## Monitoring

- [ ] Log all AI queries with timing
- [ ] Track token usage metrics
- [ ] Monitor embedding generation success rate
- [ ] Set up alerts for AI service failures
- [ ] Track feedback sentiment

---

## Documentation

- [ ] Document AI query types and examples
- [ ] Document insight types and triggers
- [ ] Document token limits by plan
- [ ] Add troubleshooting guide

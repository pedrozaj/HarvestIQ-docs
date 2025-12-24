# Phase 7: AI Integration

Conversational AI interface for querying project data.

---

## Phase Goal

Build a natural language query system allowing users to ask questions about their projects, budgets, schedules, and documents.

---

## Dependencies

- Phase 6 complete (needs all data available)

---

## Deliverables

1. AI chat page with conversation history
2. Floating AI panel on all pages
3. Natural language query processing
4. Document RAG (Retrieval Augmented Generation)
5. Proactive AI insights
6. Token usage tracking and limits

---

## Tables Introduced

| Table | Purpose |
|-------|---------|
| `ai_conversations` | Chat sessions |
| `ai_messages` | Individual messages |
| `ai_feedback` | User feedback on responses |
| `ai_insights` | Proactive AI insights |
| `ai_patterns` | Learned baselines |
| `document_embeddings` | Vector embeddings for RAG |

---

## Endpoints Introduced

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET/POST | `/ai/conversations` | Manage conversations |
| POST | `/ai/conversations/:id/messages` | Send message |
| POST | `/ai/feedback` | Submit feedback |
| GET | `/ai/insights` | Get insights |
| POST | `/ai/insights/:id/dismiss` | Dismiss insight |
| GET | `/ai/suggestions` | Get suggestions |

---

## UI Pages Introduced

| Route | Page | Purpose |
|-------|------|---------|
| /ai | AI Chat | Full chat interface |
| Header | AI Panel | Floating chat panel |

---

## Acceptance Criteria

- [ ] Users can ask natural language questions
- [ ] AI retrieves and analyzes data correctly
- [ ] Conversation history persists
- [ ] Document content is searchable
- [ ] Insights generate automatically
- [ ] Token limits are enforced

---

## Related Documentation

- [SCHEMA.md](./SCHEMA.md) | [API.md](./API.md) | [UI.md](./UI.md) | [PIPELINE.md](./PIPELINE.md) | [CHECKLIST.md](./CHECKLIST.md)

# Phase 7: API Endpoints

AI conversation and insights endpoints.

---

## Conversations

### GET /ai/conversations

List user's conversations.

```typescript
Query: {
  limit?: number;
  offset?: number;
}

Response: {
  data: [{
    id: string;
    title: string;
    project: { id, name } | null;
    createdAt: string;
    updatedAt: string;
  }]
}
```

### POST /ai/conversations

Start new conversation.

```typescript
Request: {
  projectId?: string;  // Optional project context
}

Response: {
  data: {
    id: string;
    title: string | null;
    createdAt: string;
  }
}
```

### GET /ai/conversations/:id

Get conversation with messages.

```typescript
Response: {
  data: {
    id: string;
    title: string;
    project: { id, name } | null;
    messages: [{
      id: string;
      role: "user" | "assistant";
      content: string;
      queryType: string | null;
      sources: [{
        type: string;
        id: string;
        name: string;
      }] | null;
      createdAt: string;
    }];
    createdAt: string;
  }
}
```

### DELETE /ai/conversations/:id

Delete conversation.

---

## Messages

### POST /ai/conversations/:id/messages

Send message and get AI response.

```typescript
Request: {
  message: string;
  context?: {
    projectId?: string;
  };
}

Response: {
  data: {
    id: string;
    role: "assistant";
    content: string;
    queryType: string;
    sources: [{
      type: string;
      id: string;
      name: string;
    }] | null;
    tokensUsed: number;
  }
}
```

**Errors**

| Code | Status | Message |
|------|--------|---------|
| BIZ_5004 | 429 | Monthly AI query limit reached |

---

## Feedback

### POST /ai/feedback

Submit feedback on AI response.

```typescript
Request: {
  messageId: string;
  rating: number;        // 1 (thumbs down) or 5 (thumbs up)
  feedbackText?: string;
}

Response: {
  data: {
    id: string;
    createdAt: string;
  }
}
```

---

## Insights

### GET /ai/insights

Get active insights.

```typescript
Query: {
  projectId?: string;
  insightType?: string;
  isDismissed?: boolean;
  limit?: number;
}

Response: {
  data: [{
    id: string;
    type: string;
    title: string;
    description: string;
    severity: string;
    project: { id, name } | null;
    data: object | null;
    createdAt: string;
  }]
}
```

### POST /ai/insights/:id/dismiss

Dismiss insight.

```typescript
Response: {
  data: {
    id: string;
    isDismissed: true;
    dismissedAt: string;
  }
}
```

---

## Suggestions

### GET /ai/suggestions

Get contextual suggested questions.

```typescript
Query: {
  projectId?: string;
}

Response: {
  data: {
    suggestions: string[];
  }
}
```

**Example Response:**

```json
{
  "data": {
    "suggestions": [
      "What tasks are overdue?",
      "How's the budget looking?",
      "Any upcoming milestones?"
    ]
  }
}
```

---

## Token Usage

Returned in user profile:

```typescript
// GET /users/me includes:
{
  builder: {
    aiTokensUsedMonth: number;
    aiTokensLimit: number;
    aiTokensResetAt: string;
  }
}
```

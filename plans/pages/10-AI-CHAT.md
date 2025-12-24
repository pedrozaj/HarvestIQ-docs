# AI Chat

Full-screen conversational AI interface for querying project data.

Route: `/ai`

---

## Full Page Layout

```
+-----------------------------------------------------------------------------------+
| [Logo] HarvestIQ     [_____Search_____]              [AI] [Bell(3)] [Avatar]      |
+-----------------------------------------------------------------------------------+
|          |                                                                        |
| Dashboard|  +------------------+  +--------------------------------------------+  |
| Projects |  | CONVERSATIONS    |  |                                            |  |
| Tasks    |  +------------------+  |  AI ASSISTANT                              |  |
|          |  |                  |  |                                            |  |
|          |  | [+ New Chat]     |  |  Context: All Projects                [v]  |  |
| ──────── |  |                  |  |                                            |  |
| Team     |  | Today            |  +--------------------------------------------+  |
| Settings |  | > Plumbing costs |  |                                            |  |
|          |  | > Schedule check |  |  [Bot Avatar]                              |  |
|          |  |                  |  |  Hello! I can help you query your project  |  |
|          |  | Yesterday        |  |  data. Try asking me things like:          |  |
|          |  | > Budget review  |  |                                            |  |
|          |  | > Cost per unit  |  |  - "How much did I spend on plumbing?"     |  |
|          |  |                  |  |  - "Where are my bottlenecks?"             |  |
|          |  | Last Week        |  |  - "Which project is most efficient?"      |  |
|          |  | > Project summary|  |  - "What invoices are overdue?"            |  |
|          |  | > Milestone check|  |                                            |  |
|          |  |                  |  +--------------------------------------------+  |
|          |  |                  |  |                                            |  |
|          |  |                  |  |  [User Avatar]                             |  |
|          |  |                  |  |  How much did I spend on plumbing?         |  |
|          |  |                  |  |                                            |  |
|          |  |                  |  +--------------------------------------------+  |
|          |  |                  |  |                                            |  |
|          |  |                  |  |  [Bot Avatar]                              |  |
|          |  |                  |  |  You've spent $145,230 on plumbing across  |  |
|          |  |                  |  |  all projects:                             |  |
|          |  |                  |  |                                            |  |
|          |  |                  |  |  - Riverside Apartments: $45,230           |  |
|          |  |                  |  |  - Oak Grove Homes: $52,000                |  |
|          |  |                  |  |  - Summit Plaza: $48,000                   |  |
|          |  |                  |  |                                            |  |
|          |  |                  |  |  Riverside is 12% over your typical        |  |
|          |  |                  |  |  plumbing budget.                          |  |
|          |  |                  |  |                                            |  |
|          |  |                  |  |  [Thumbs Up] [Thumbs Down]                 |  |
|          |  |                  |  |                                            |  |
|          |  +------------------+  +--------------------------------------------+  |
|          |                        |                                            |  |
|          |                        | [Type your question...              ] [>]  |  |
|          |                        +--------------------------------------------+  |
|          |                                                                        |
+-----------------------------------------------------------------------------------+
```

---

## Conversation Sidebar

### Elements

| Element | Description |
|---------|-------------|
| + New Chat | Start fresh conversation |
| Conversation list | Grouped by date |
| Conversation title | Auto-generated from first message |

### API Needed

```
GET /ai/conversations?limit=20&offset=0

Response:
{
  data: [{
    id: uuid,
    title: string,
    project: { id, name },  // If scoped to project
    created_at: timestamp,
    updated_at: timestamp
  }]
}

POST /ai/conversations
Body: { project_id?: uuid }

Response: { id, title, created_at }

DELETE /ai/conversations/:id
```

---

## Chat Area

### Context Selector

Dropdown to scope AI queries:

```
+-------------------------+
| All Projects            |
+-------------------------+
| Riverside Apartments    |
| Oak Grove Homes         |
| Summit Plaza            |
| Lakefront Condos        |
+-------------------------+
```

When a project is selected, AI automatically scopes queries to that project.

### Message Types

| Type | Styling |
|------|---------|
| User message | Right-aligned, user avatar |
| Bot response | Left-aligned, bot avatar |
| Sources | Collapsible list of data sources |
| Feedback | Thumbs up/down on bot responses |

---

## Message Input

```
+--------------------------------------------------------------------+
| [Type your question...                                      ] [>]  |
+--------------------------------------------------------------------+
```

### Suggested Questions (empty state or after response)

```
+--------------------------------------------------------------------+
| Suggestions:                                                       |
| [What are my overdue tasks?] [Show budget summary] [Any concerns?] |
+--------------------------------------------------------------------+
```

### API Needed

```
POST /ai/conversations/:id/messages
Body: {
  message: string,
  context?: {
    projectId?: uuid
  }
}

Response:
{
  id: uuid,
  role: "assistant",
  content: string,
  query_type: string,
  sources: [{
    type: string,
    id: uuid,
    name: string
  }],
  tokens_used: number
}
```

---

## Response with Sources

When AI references specific data:

```
+--------------------------------------------------------------------+
|  [Bot Avatar]                                                      |
|  I found 3 bottlenecks in your projects:                           |
|                                                                    |
|  1. Riverside - Electrical rough-in blocked by framing inspection  |
|     (5 days overdue)                                               |
|  2. Oak Grove - Cabinet installation waiting on countertop delivery|
|     (delayed 2 weeks)                                              |
|  3. Summit - Final inspection pending permit approval              |
|                                                                    |
|  The framing inspection is most critical - it's blocking 4 tasks.  |
|                                                                    |
|  Sources: [v]                                                      |
|  +--------------------------------------------------------------+  |
|  | Schedule Task: Framing Inspection (Riverside)                |  |
|  | Schedule Task: Electrical Rough-in (Riverside)               |  |
|  | Schedule Task: Cabinet Installation (Oak Grove)              |  |
|  +--------------------------------------------------------------+  |
|                                                                    |
|  [Thumbs Up] [Thumbs Down]                                         |
+--------------------------------------------------------------------+
```

---

## Feedback

```
POST /ai/feedback
Body: {
  message_id: uuid,
  rating: number,  // 1 (thumbs down) or 5 (thumbs up)
  feedback_text?: string
}
```

---

## Floating AI Panel

Available on all pages via the AI button in header.

```
                                        +---------------------------+
                                        | AI Assistant        [_][X]|
                                        +---------------------------+
                                        | Context: Riverside Apts   |
                                        +---------------------------+
                                        |                           |
                                        | [Bot]: How can I help     |
                                        |        with Riverside?    |
                                        |                           |
                                        | [You]: Am I over budget   |
                                        |        on electrical?     |
                                        |                           |
                                        | [Bot]: No, you're         |
                                        |        currently at       |
                                        |        $38,500 of your    |
                                        |        $58,000 budget     |
                                        |        (66%). You have    |
                                        |        $19,500 remaining. |
                                        |                           |
                                        +---------------------------+
                                        | [Ask a question...] [>]   |
                                        +---------------------------+
                                        | [Open Full Chat]          |
                                        +---------------------------+
```

### Behavior

- Inherits context from current page (if on project, scopes to that project)
- Conversation persists during session
- "Open Full Chat" navigates to `/ai` with current conversation
- Minimizes to header button

---

## Query Types Supported

| Query Type | Example | Data Sources |
|------------|---------|--------------|
| Spending | "How much on plumbing?" | payments, budget_items |
| Schedule | "Where are bottlenecks?" | schedule_tasks, dependencies |
| Budget | "Am I over budget?" | budget_items, payments |
| Comparison | "Which project is most efficient?" | all projects |
| Trend | "Spending trend last 6 months?" | payments by date |
| Document | "What does invoice #4521 say?" | documents, embeddings |
| Unit Analysis | "Cost per unit on Riverside?" | budget, units |
| Open-ended | "Any concerns with Riverside?" | all data |

---

## AI Suggestions Endpoint

For suggested follow-up questions:

```
GET /ai/suggestions?project_id=X

Response:
{
  suggestions: [
    "What tasks are overdue?",
    "How's the budget looking?",
    "Any upcoming milestones?"
  ]
}
```

---

## Token Limit Warning

When approaching monthly AI token limit:

```
+--------------------------------------------------------------------+
| [Warning Icon] You've used 85% of your monthly AI queries.        |
| Upgrade your plan for unlimited AI access.              [Upgrade] |
+--------------------------------------------------------------------+
```

When limit reached:

```
+--------------------------------------------------------------------+
| [Error Icon] Monthly AI query limit reached.                      |
| Your limit resets on January 1st.                       [Upgrade] |
+--------------------------------------------------------------------+
```

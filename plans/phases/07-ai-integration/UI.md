# Phase 7: UI Pages

AI chat interface and floating panel.

---

## AI Chat Page

Route: `/ai`

```
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
|          |  |                  |  |                                            |  |
|          |  +------------------+  +--------------------------------------------+  |
|          |                        |                                            |  |
|          |                        |  [User Avatar]                             |  |
|          |                        |  How much did I spend on plumbing?         |  |
|          |                        |                                            |  |
|          |                        +--------------------------------------------+  |
|          |                        |                                            |  |
|          |                        |  [Bot Avatar]                              |  |
|          |                        |  You've spent $145,230 on plumbing across  |  |
|          |                        |  all projects:                             |  |
|          |                        |                                            |  |
|          |                        |  - Riverside Apartments: $45,230           |  |
|          |                        |  - Oak Grove Homes: $52,000                |  |
|          |                        |  - Summit Plaza: $48,000                   |  |
|          |                        |                                            |  |
|          |                        |  Riverside is 12% over your typical        |  |
|          |                        |  plumbing budget.                          |  |
|          |                        |                                            |  |
|          |                        |  Sources: [v]                              |  |
|          |                        |  +----------------------------------------+ |  |
|          |                        |  | Payment: Plumbing rough-in (Riverside) | |  |
|          |                        |  | Payment: Final plumbing (Oak Grove)    | |  |
|          |                        |  +----------------------------------------+ |  |
|          |                        |                                            |  |
|          |                        |  [Thumbs Up] [Thumbs Down]                 |  |
|          |                        |                                            |  |
|          |                        +--------------------------------------------+  |
|          |                        |                                            |  |
|          |                        | [Type your question...              ] [>]  |  |
|          |                        +--------------------------------------------+  |
|          |                        | Suggestions: [What tasks are overdue?]     |  |
|          |                        |              [Show budget summary]         |  |
|          |                        +--------------------------------------------+  |
+-----------------------------------------------------------------------------------+
```

---

## Floating AI Panel

Available via AI button in header.

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

### Panel Behavior

- Inherits context from current page (project if on project page)
- Minimizes to header button
- Persists conversation during session
- "Open Full Chat" links to /ai with current conversation

---

## Token Limit Warning

When approaching limit:

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

---

## Components to Build

| Component | Purpose |
|-----------|---------|
| ConversationList | Sidebar conversation list |
| ConversationItem | Individual conversation row |
| ChatArea | Main chat message area |
| ChatMessage | User or assistant message |
| MessageInput | Input with submit |
| ContextSelector | Project dropdown |
| SourcesList | Collapsible sources |
| FeedbackButtons | Thumbs up/down |
| SuggestionChips | Clickable suggestions |
| AIPanel | Floating panel |
| TokenWarning | Limit warning banner |

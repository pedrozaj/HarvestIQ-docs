# Phase 5: API Endpoints

Task items, reminders, and notification endpoints.

---

## Task Items

### GET /task-items

List task items for current user or project.

```typescript
Query: {
  projectId?: string;
  status?: string;
  priority?: string;
  assignedTo?: string;  // 'me' for current user
  overdue?: boolean;
  limit?: number;
  offset?: number;
}

Response: {
  data: [{
    id: string;
    title: string;
    description: string | null;
    project: { id, name } | null;
    scheduleTask: { id, name } | null;
    assignedTo: { id, name } | null;
    status: string;
    priority: string;
    dueDate: string | null;
    isOverdue: boolean;
    createdAt: string;
  }],
  total: number
}
```

### POST /task-items

Create task item.

```typescript
Request: {
  title: string;
  description?: string;
  projectId?: string;
  scheduleTaskId?: string;
  assignedTo?: string;
  priority?: string;
  dueDate?: string;
}
```

### GET /task-items/:id

Get task item details.

### PUT /task-items/:id

Update task item.

### DELETE /task-items/:id

Delete task item.

### PUT /task-items/:id/complete

Mark task item as completed.

```typescript
Response: {
  data: {
    id: string;
    status: "completed";
    completedAt: string;
    completedBy: { id, name };
  }
}
```

---

## Reminders

### GET /reminders

List user's reminders.

```typescript
Query: {
  upcoming?: boolean;   // Only future reminders
  dismissed?: boolean;  // Include dismissed
  limit?: number;
  offset?: number;
}

Response: {
  data: [{
    id: string;
    title: string;
    description: string | null;
    remindAt: string;
    recurrence: string;
    relatedEntity: { type, id, name } | null;
    isDismissed: boolean;
  }]
}
```

### POST /reminders

Create reminder.

```typescript
Request: {
  title: string;
  description?: string;
  remindAt: string;       // ISO datetime
  recurrence?: string;    // none, daily, weekly, monthly
  projectId?: string;
  relatedEntityType?: string;
  relatedEntityId?: string;
}
```

### PUT /reminders/:id

Update reminder.

### DELETE /reminders/:id

Delete reminder.

### PUT /reminders/:id/dismiss

Dismiss reminder.

---

## Notifications

### GET /notifications

List user's notifications.

```typescript
Query: {
  unreadOnly?: boolean;
  type?: string;
  limit?: number;
  offset?: number;
}

Response: {
  data: [{
    id: string;
    type: string;
    title: string;
    message: string;
    data: object | null;
    isRead: boolean;
    createdAt: string;
  }],
  total: number;
  unreadCount: number;
}
```

### PATCH /notifications/:id/read

Mark notification as read.

### POST /notifications/read-all

Mark all notifications as read.

### DELETE /notifications/:id

Delete notification.

---

## Notification Preferences

### GET /users/me/notification-preferences

Get user's notification preferences.

```typescript
Response: {
  data: [{
    notificationType: string;
    emailEnabled: boolean;
    smsEnabled: boolean;
    inAppEnabled: boolean;
  }]
}
```

### PUT /users/me/notification-preferences

Update preferences.

```typescript
Request: {
  preferences: [{
    notificationType: string;
    emailEnabled?: boolean;
    smsEnabled?: boolean;
    inAppEnabled?: boolean;
  }]
}
```

---

## Alert Thresholds

### GET /alert-thresholds

List alert thresholds.

```typescript
Query: { projectId?: string }

Response: {
  data: [{
    id: string;
    project: { id, name } | null;
    category: { id, name } | null;
    thresholdType: string;
    thresholdValue: number;
    isActive: boolean;
  }]
}
```

### POST /alert-thresholds

Create threshold.

```typescript
Request: {
  projectId?: string;
  categoryId?: string;
  thresholdType: "percentage" | "amount";
  thresholdValue: number;
}
```

### PUT /alert-thresholds/:id

Update threshold.

### DELETE /alert-thresholds/:id

Delete threshold.

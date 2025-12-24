# Phase 5: Background Jobs

Job queue configuration and worker implementations using pg-boss.

---

## pg-boss Setup

```typescript
// src/jobs/queue.ts
import PgBoss from 'pg-boss';

const boss = new PgBoss(process.env.DATABASE_URL);

await boss.start();

// Register job handlers
boss.work('reminder-check', reminderCheckHandler);
boss.work('due-date-notifications', dueDateHandler);
boss.work('budget-alert-check', budgetAlertHandler);
boss.work('send-email', emailHandler);
boss.work('send-notification', notificationHandler);

// Schedule recurring jobs
await boss.schedule('reminder-check', '* * * * *');        // Every minute
await boss.schedule('due-date-notifications', '0 8 * * *'); // Daily at 8 AM
```

---

## Job: reminder-check

Runs every minute to check for due reminders.

```typescript
// src/jobs/reminder.job.ts
async function reminderCheckHandler(job: Job) {
  const now = new Date();
  const upcoming = new Date(now.getTime() + 60000); // Next minute

  // Find reminders due in the next minute
  const reminders = await db.query(`
    SELECT r.*, u.email, u.name as user_name
    FROM reminders r
    JOIN users u ON r.user_id = u.id
    WHERE r.remind_at BETWEEN $1 AND $2
    AND r.is_dismissed = false
  `, [now, upcoming]);

  for (const reminder of reminders.rows) {
    // Create notification
    await createNotification({
      builderId: reminder.builder_id,
      userId: reminder.user_id,
      type: 'reminder',
      title: 'Reminder',
      message: reminder.title,
      data: {
        reminderId: reminder.id,
        entityType: reminder.related_entity_type,
        entityId: reminder.related_entity_id,
      },
    });

    // Handle recurrence
    if (reminder.recurrence !== 'none') {
      await scheduleNextOccurrence(reminder);
    }
  }
}
```

---

## Job: due-date-notifications

Runs daily to send due date reminders.

```typescript
// src/jobs/due-dates.job.ts
async function dueDateHandler(job: Job) {
  const today = new Date();
  const tomorrow = addDays(today, 1);
  const nextWeek = addDays(today, 7);

  // Task items due tomorrow
  const dueTasks = await db.query(`
    SELECT ti.*, u.email, p.name as project_name
    FROM task_items ti
    JOIN users u ON ti.assigned_to = u.id
    LEFT JOIN projects p ON ti.project_id = p.id
    WHERE ti.due_date = $1
    AND ti.status != 'completed'
  `, [format(tomorrow, 'yyyy-MM-dd')]);

  for (const task of dueTasks.rows) {
    await createNotification({
      userId: task.assigned_to,
      type: 'task_due',
      title: 'Task Due Tomorrow',
      message: `"${task.title}" is due tomorrow`,
      data: { taskItemId: task.id, projectId: task.project_id },
    });
  }

  // Milestones approaching (7 days)
  const upcomingMilestones = await db.query(`
    SELECT m.*, p.name as project_name
    FROM schedule_milestones m
    JOIN projects p ON m.project_id = p.id
    WHERE m.target_date = $1
    AND m.status = 'upcoming'
  `, [format(nextWeek, 'yyyy-MM-dd')]);

  // ... create notifications for milestone stakeholders

  // Overdue invoices
  const overdueInvoices = await db.query(`
    SELECT i.*, p.name as project_name
    FROM invoices i
    JOIN projects p ON i.project_id = p.id
    WHERE i.due_date < $1
    AND i.status != 'paid'
    AND i.deleted_at IS NULL
  `, [format(today, 'yyyy-MM-dd')]);

  // ... create notifications for admins
}
```

---

## Job: budget-alert-check

Triggered when payment is recorded.

```typescript
// src/jobs/budget-alert.job.ts
async function budgetAlertHandler(job: Job) {
  const { projectId, categoryId, builderId } = job.data;

  // Get budget summary for category
  const summary = await db.query(`
    SELECT
      COALESCE(SUM(bi.estimated_amount), 0) as estimated,
      COALESCE(SUM(bi.actual_amount), 0) as actual
    FROM budget_items bi
    WHERE bi.project_id = $1
    AND bi.category_id = $2
  `, [projectId, categoryId]);

  const { estimated, actual } = summary.rows[0];
  const percentUsed = estimated > 0 ? (actual / estimated) * 100 : 0;

  // Check thresholds
  const thresholds = await db.query(`
    SELECT * FROM alert_thresholds
    WHERE (project_id = $1 OR project_id IS NULL)
    AND (category_id = $2 OR category_id IS NULL)
    AND is_active = true
    AND builder_id = $3
  `, [projectId, categoryId, builderId]);

  for (const threshold of thresholds.rows) {
    let triggered = false;

    if (threshold.threshold_type === 'percentage') {
      triggered = percentUsed >= threshold.threshold_value;
    } else {
      triggered = actual >= threshold.threshold_value;
    }

    if (triggered && !recentlyTriggered(threshold)) {
      // Create budget alert notification for admins
      await createBudgetAlert(projectId, categoryId, threshold);
      await updateThresholdTriggered(threshold.id);
    }
  }
}
```

---

## Job: send-email

Email delivery worker.

```typescript
// src/jobs/email.job.ts
async function emailHandler(job: Job) {
  const { to, subject, template, data, deliveryId } = job.data;

  try {
    await emailService.send({
      to,
      subject,
      template,
      data,
    });

    await markDeliverySent(deliveryId);
  } catch (error) {
    await markDeliveryFailed(deliveryId, error.message);

    // Retry if under max attempts
    const delivery = await getDelivery(deliveryId);
    if (delivery.attempts < delivery.max_attempts) {
      throw error; // pg-boss will retry
    }
  }
}
```

---

## Job: send-notification

Create notification and queue deliveries.

```typescript
// src/jobs/notification.job.ts
async function notificationHandler(job: Job) {
  const { builderId, userId, type, title, message, data } = job.data;

  // Create notification record
  const notification = await db.query(`
    INSERT INTO notifications (builder_id, user_id, type, title, message, data)
    VALUES ($1, $2, $3, $4, $5, $6)
    RETURNING *
  `, [builderId, userId, type, title, message, data]);

  // Get user preferences
  const prefs = await getNotificationPreferences(userId, type);

  // Create delivery records
  if (prefs.inAppEnabled) {
    await createDelivery(notification.id, 'in_app');
    // In-app is instant, mark sent immediately
  }

  if (prefs.emailEnabled) {
    const delivery = await createDelivery(notification.id, 'email');
    await boss.send('send-email', {
      to: user.email,
      subject: title,
      template: `notification-${type}`,
      data: { title, message, ...data },
      deliveryId: delivery.id,
    });
  }
}
```

---

## Notification Flow

```
Event Occurs (task assigned, payment recorded, etc.)
    ↓
Queue 'send-notification' job
    ↓
Create notification record
    ↓
Check user preferences
    ↓
Create delivery records per enabled channel
    ↓
Queue 'send-email' job (if email enabled)
    ↓
Email worker sends via Resend/SendGrid
    ↓
Update delivery status
```

---

## Email Templates

| Template | Subject | Variables |
|----------|---------|-----------|
| task-assigned | Task Assigned: {title} | title, assignedBy, projectName |
| task-due | Task Due Tomorrow | title, dueDate, projectName |
| milestone-approaching | Milestone in {days} Days | milestoneName, targetDate, projectName |
| invoice-due | Invoice Due: {invoiceNumber} | invoiceNumber, amount, dueDate |
| budget-alert | Budget Alert: {category} | categoryName, percentUsed, projectName |
| reminder | Reminder: {title} | title, description |

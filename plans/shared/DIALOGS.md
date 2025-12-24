# Dialogs, Modals & Alerts

Centralized patterns for all user interaction dialogs across the application.

---

## Dialog Types

| Type | Purpose | User Action |
|------|---------|-------------|
| Modal | Forms, detailed views | Submit or Cancel |
| Confirmation | Destructive actions | Confirm or Cancel |
| Alert | Important information | Acknowledge |
| Toast | Transient feedback | Auto-dismiss or Dismiss |
| Sheet | Mobile-friendly panels | Close or Submit |

---

## Component Library

Use shadcn/ui dialog components as the base:

```
src/components/ui/
├── dialog.tsx          (base dialog)
├── alert-dialog.tsx    (confirmation dialogs)
├── sheet.tsx           (slide-out panels)
└── toast.tsx           (notifications)

src/components/dialogs/
├── ConfirmDialog.tsx   (reusable confirmation)
├── FormDialog.tsx      (reusable form wrapper)
└── AlertBanner.tsx     (page-level alerts)
```

---

## Confirmation Dialog Pattern

### Component: `ConfirmDialog.tsx`

```typescript
interface ConfirmDialogProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  title: string;
  description: string;
  confirmLabel?: string;
  cancelLabel?: string;
  variant?: 'default' | 'destructive';
  onConfirm: () => void | Promise<void>;
  loading?: boolean;
}

export function ConfirmDialog({
  open,
  onOpenChange,
  title,
  description,
  confirmLabel = 'Confirm',
  cancelLabel = 'Cancel',
  variant = 'default',
  onConfirm,
  loading = false,
}: ConfirmDialogProps) {
  return (
    <AlertDialog open={open} onOpenChange={onOpenChange}>
      <AlertDialogContent>
        <AlertDialogHeader>
          <AlertDialogTitle>{title}</AlertDialogTitle>
          <AlertDialogDescription>{description}</AlertDialogDescription>
        </AlertDialogHeader>
        <AlertDialogFooter>
          <AlertDialogCancel disabled={loading}>
            {cancelLabel}
          </AlertDialogCancel>
          <AlertDialogAction
            onClick={onConfirm}
            disabled={loading}
            className={variant === 'destructive' ? 'bg-destructive' : ''}
          >
            {loading ? 'Processing...' : confirmLabel}
          </AlertDialogAction>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  );
}
```

### Standard Confirmations

| Action | Title | Description | Confirm Label | Variant |
|--------|-------|-------------|---------------|---------|
| Delete Project | Delete Project | This will archive the project and all its data. This action cannot be undone. | Delete | destructive |
| Remove Member | Remove Team Member | Remove {name} from {org}? They will lose access to all projects. | Remove | destructive |
| Change Role | Change Role | Change {name}'s role to {role}? | Change Role | default |
| Mark Complete | Mark Complete | Mark this task as completed? | Complete | default |
| Cancel Invitation | Cancel Invitation | Cancel the invitation to {email}? | Cancel Invitation | destructive |
| Delete Document | Delete Document | Delete {filename}? This cannot be undone. | Delete | destructive |
| Dismiss Insight | Dismiss Insight | Dismiss this insight? It won't appear again. | Dismiss | default |

---

## Form Modal Pattern

### Component: `FormDialog.tsx`

```typescript
interface FormDialogProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  title: string;
  description?: string;
  children: React.ReactNode;
  onSubmit: () => void | Promise<void>;
  submitLabel?: string;
  loading?: boolean;
  size?: 'sm' | 'md' | 'lg' | 'xl';
}

const sizeClasses = {
  sm: 'max-w-sm',
  md: 'max-w-md',
  lg: 'max-w-lg',
  xl: 'max-w-xl',
};

export function FormDialog({
  open,
  onOpenChange,
  title,
  description,
  children,
  onSubmit,
  submitLabel = 'Save',
  loading = false,
  size = 'md',
}: FormDialogProps) {
  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent className={sizeClasses[size]}>
        <DialogHeader>
          <DialogTitle>{title}</DialogTitle>
          {description && (
            <DialogDescription>{description}</DialogDescription>
          )}
        </DialogHeader>
        <form onSubmit={(e) => { e.preventDefault(); onSubmit(); }}>
          <div className="py-4 space-y-4">
            {children}
          </div>
          <DialogFooter>
            <Button
              type="button"
              variant="outline"
              onClick={() => onOpenChange(false)}
              disabled={loading}
            >
              Cancel
            </Button>
            <Button type="submit" disabled={loading}>
              {loading ? 'Saving...' : submitLabel}
            </Button>
          </DialogFooter>
        </form>
      </DialogContent>
    </Dialog>
  );
}
```

### Standard Form Modals

| Form | Title | Size | Submit Label |
|------|-------|------|--------------|
| Create Project | New Project | lg | Create Project |
| Edit Project | Edit Project | lg | Save Changes |
| Create Task | New Task | md | Create Task |
| Create Budget Item | Add Budget Item | md | Add Item |
| Upload Document | Upload Document | md | Upload |
| Record Payment | Record Payment | md | Record Payment |
| Invite Member | Invite Team Member | sm | Send Invitation |
| Create Reminder | New Reminder | sm | Create Reminder |

---

## Toast Notifications

### Toast Hook

```typescript
import { useToast } from '@/components/ui/use-toast';

// Usage
const { toast } = useToast();

// Success
toast({
  title: 'Project created',
  description: 'Riverside Apartments has been created.',
});

// Error
toast({
  title: 'Error',
  description: 'Failed to save changes. Please try again.',
  variant: 'destructive',
});

// With action
toast({
  title: 'Task completed',
  description: 'Framing inspection marked complete.',
  action: (
    <ToastAction altText="Undo" onClick={handleUndo}>
      Undo
    </ToastAction>
  ),
});
```

### Standard Toast Messages

| Action | Title | Description |
|--------|-------|-------------|
| Create success | {Entity} created | {Name} has been created. |
| Update success | Changes saved | Your changes have been saved. |
| Delete success | {Entity} deleted | {Name} has been deleted. |
| Error | Error | {Error message}. Please try again. |
| Upload success | File uploaded | {Filename} has been uploaded. |
| Email sent | Invitation sent | Invitation sent to {email}. |
| Copy success | Copied | Link copied to clipboard. |

---

## Alert Banners

### Component: `AlertBanner.tsx`

```typescript
interface AlertBannerProps {
  type: 'info' | 'warning' | 'error' | 'success';
  title?: string;
  message: string;
  action?: {
    label: string;
    onClick: () => void;
  };
  dismissible?: boolean;
  onDismiss?: () => void;
}

const alertStyles = {
  info: 'bg-blue-50 border-blue-200 text-blue-800',
  warning: 'bg-yellow-50 border-yellow-200 text-yellow-800',
  error: 'bg-red-50 border-red-200 text-red-800',
  success: 'bg-green-50 border-green-200 text-green-800',
};

const alertIcons = {
  info: InfoIcon,
  warning: AlertTriangleIcon,
  error: XCircleIcon,
  success: CheckCircleIcon,
};
```

### Standard Alert Banners

| Context | Type | Message |
|---------|------|---------|
| AI Token Limit | warning | You've used 85% of your monthly AI queries. |
| AI Limit Reached | error | Monthly AI query limit reached. Resets on {date}. |
| Over Budget | warning | This project is {percent}% over budget. |
| Overdue Tasks | warning | {count} tasks are overdue. |
| Session Expiring | warning | Your session will expire in 5 minutes. |
| Maintenance | info | Scheduled maintenance on {date} at {time}. |

---

## Loading States

### In Dialogs

```typescript
// Submit button loading state
<Button type="submit" disabled={loading}>
  {loading ? (
    <>
      <Loader2 className="mr-2 h-4 w-4 animate-spin" />
      Saving...
    </>
  ) : (
    'Save'
  )}
</Button>

// Dialog with loading overlay
{loading && (
  <div className="absolute inset-0 bg-background/80 flex items-center justify-center">
    <Loader2 className="h-8 w-8 animate-spin" />
  </div>
)}
```

### Loading Text Patterns

| Action | Loading Text |
|--------|--------------|
| Create | Creating... |
| Save | Saving... |
| Delete | Deleting... |
| Upload | Uploading... |
| Send | Sending... |
| Process | Processing... |

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| Escape | Close dialog/modal |
| Enter | Submit form (when in input) |
| Tab | Navigate form fields |
| Cmd/Ctrl + Enter | Submit form (anywhere) |

---

## Accessibility Requirements

1. **Focus management** - Focus moves to dialog on open, returns on close
2. **Screen reader** - Use `aria-labelledby` and `aria-describedby`
3. **Keyboard navigation** - All actions accessible via keyboard
4. **Focus trap** - Focus stays within dialog while open
5. **Escape to close** - All dialogs close on Escape key

```typescript
// Focus management example
useEffect(() => {
  if (open) {
    // Store current focus
    previousFocusRef.current = document.activeElement;
    // Focus first input or close button
    firstInputRef.current?.focus();
  } else {
    // Restore focus
    previousFocusRef.current?.focus();
  }
}, [open]);
```

---

## Mobile Considerations

1. **Use Sheet for forms** - Bottom sheet pattern for mobile forms
2. **Full-width buttons** - Action buttons span full width on mobile
3. **Larger touch targets** - Minimum 44x44px for buttons
4. **Swipe to dismiss** - Support swipe down to close sheets

```typescript
// Responsive dialog/sheet
function ResponsiveDialog({ children, ...props }) {
  const isMobile = useMediaQuery('(max-width: 640px)');

  if (isMobile) {
    return <Sheet {...props}>{children}</Sheet>;
  }

  return <Dialog {...props}>{children}</Dialog>;
}
```

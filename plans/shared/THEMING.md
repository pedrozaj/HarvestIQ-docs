# Theming & Styling

Centralized design tokens and styling standards for consistent UI across the application.

---

## Design System Foundation

Built on Tailwind CSS with shadcn/ui component library.

---

## Color Palette

### Location: `tailwind.config.ts`

```typescript
const config = {
  theme: {
    extend: {
      colors: {
        // Brand colors
        brand: {
          50: '#f0fdf4',
          100: '#dcfce7',
          200: '#bbf7d0',
          300: '#86efac',
          400: '#4ade80',
          500: '#22c55e',  // Primary brand
          600: '#16a34a',
          700: '#15803d',
          800: '#166534',
          900: '#14532d',
        },

        // Semantic colors (using CSS variables for dark mode)
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',

        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
      },
    },
  },
};
```

### CSS Variables: `globals.css`

```css
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 142.1 76.2% 36.3%;
    --primary-foreground: 355.7 100% 97.3%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 142.1 76.2% 36.3%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;
    --primary: 142.1 70.6% 45.3%;
    --primary-foreground: 144.9 80.4% 10%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 142.1 76.2% 36.3%;
  }
}
```

---

## Status Colors

### Location: `src/lib/theme.ts`

```typescript
export const STATUS_COLORS = {
  // Project status
  project: {
    planning: { bg: 'bg-blue-100', text: 'text-blue-800', border: 'border-blue-200' },
    active: { bg: 'bg-green-100', text: 'text-green-800', border: 'border-green-200' },
    on_hold: { bg: 'bg-yellow-100', text: 'text-yellow-800', border: 'border-yellow-200' },
    completed: { bg: 'bg-gray-100', text: 'text-gray-800', border: 'border-gray-200' },
  },

  // Task status
  task: {
    not_started: { bg: 'bg-gray-100', text: 'text-gray-800' },
    in_progress: { bg: 'bg-blue-100', text: 'text-blue-800' },
    completed: { bg: 'bg-green-100', text: 'text-green-800' },
    blocked: { bg: 'bg-red-100', text: 'text-red-800' },
  },

  // Priority
  priority: {
    low: { bg: 'bg-gray-100', text: 'text-gray-700' },
    medium: { bg: 'bg-blue-100', text: 'text-blue-700' },
    high: { bg: 'bg-orange-100', text: 'text-orange-700' },
    urgent: { bg: 'bg-red-100', text: 'text-red-700' },
  },

  // Invoice status
  invoice: {
    unpaid: { bg: 'bg-red-100', text: 'text-red-800' },
    partial: { bg: 'bg-yellow-100', text: 'text-yellow-800' },
    paid: { bg: 'bg-green-100', text: 'text-green-800' },
  },

  // Milestone status
  milestone: {
    upcoming: { bg: 'bg-blue-100', text: 'text-blue-800' },
    achieved: { bg: 'bg-green-100', text: 'text-green-800' },
    missed: { bg: 'bg-red-100', text: 'text-red-800' },
  },

  // Insight severity
  insight: {
    info: { bg: 'bg-blue-50', text: 'text-blue-800', icon: 'text-blue-500' },
    warning: { bg: 'bg-yellow-50', text: 'text-yellow-800', icon: 'text-yellow-500' },
    critical: { bg: 'bg-red-50', text: 'text-red-800', icon: 'text-red-500' },
  },
} as const;

// Helper function
export function getStatusColor(type: keyof typeof STATUS_COLORS, status: string) {
  return STATUS_COLORS[type][status as keyof typeof STATUS_COLORS[typeof type]] ?? {
    bg: 'bg-gray-100',
    text: 'text-gray-800',
  };
}
```

---

## Spacing Scale

Use Tailwind's default spacing scale consistently:

| Token | Value | Usage |
|-------|-------|-------|
| `space-1` | 4px | Tight spacing, icon gaps |
| `space-2` | 8px | Default element spacing |
| `space-3` | 12px | Form field spacing |
| `space-4` | 16px | Card padding, section gaps |
| `space-6` | 24px | Large section gaps |
| `space-8` | 32px | Page section spacing |

### Standard Patterns

```typescript
// Card padding
className="p-4 sm:p-6"

// Form field spacing
className="space-y-4"

// Section spacing
className="space-y-8"

// Grid gap
className="gap-4 sm:gap-6"

// Page container
className="px-4 sm:px-6 lg:px-8 py-8"
```

---

## Typography

### Font Stack

```css
/* In globals.css */
:root {
  --font-sans: 'Inter', system-ui, -apple-system, sans-serif;
  --font-mono: 'JetBrains Mono', 'Fira Code', monospace;
}
```

### Text Styles

| Style | Class | Usage |
|-------|-------|-------|
| Page Title | `text-2xl font-bold` | Page headings |
| Section Title | `text-lg font-semibold` | Card/section headings |
| Body | `text-sm` | Default body text |
| Small | `text-xs` | Labels, metadata |
| Muted | `text-muted-foreground` | Secondary text |

### Standard Patterns

```typescript
// Page heading
<h1 className="text-2xl font-bold tracking-tight">Dashboard</h1>

// Section heading
<h2 className="text-lg font-semibold">Recent Activity</h2>

// Card title
<h3 className="font-medium">Project Summary</h3>

// Body text
<p className="text-sm text-muted-foreground">
  Last updated 2 hours ago
</p>

// Label
<label className="text-sm font-medium">Project Name</label>

// Helper text
<span className="text-xs text-muted-foreground">Optional</span>
```

---

## Component Variants

### Button Variants

```typescript
// Primary action
<Button>Create Project</Button>

// Secondary action
<Button variant="secondary">Cancel</Button>

// Destructive action
<Button variant="destructive">Delete</Button>

// Ghost (minimal)
<Button variant="ghost">View Details</Button>

// Outline
<Button variant="outline">Export</Button>

// Link style
<Button variant="link">Learn more</Button>

// Icon button
<Button variant="ghost" size="icon">
  <PlusIcon className="h-4 w-4" />
</Button>
```

### Badge Variants

```typescript
// Status badges
<Badge variant="default">Active</Badge>
<Badge variant="secondary">Planning</Badge>
<Badge variant="destructive">Overdue</Badge>
<Badge variant="outline">Draft</Badge>

// Custom status badge
function StatusBadge({ status }: { status: string }) {
  const colors = getStatusColor('project', status);
  return (
    <span className={cn('px-2 py-1 rounded-full text-xs font-medium', colors.bg, colors.text)}>
      {PROJECT_STATUS_LABELS[status]}
    </span>
  );
}
```

---

## Layout Patterns

### Page Container

```typescript
function PageContainer({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex-1 space-y-8 p-4 sm:p-6 lg:p-8">
      {children}
    </div>
  );
}
```

### Page Header

```typescript
function PageHeader({
  title,
  description,
  action,
}: {
  title: string;
  description?: string;
  action?: React.ReactNode;
}) {
  return (
    <div className="flex items-center justify-between">
      <div>
        <h1 className="text-2xl font-bold tracking-tight">{title}</h1>
        {description && (
          <p className="text-muted-foreground">{description}</p>
        )}
      </div>
      {action}
    </div>
  );
}
```

### Card Layout

```typescript
// Standard card
<Card>
  <CardHeader>
    <CardTitle>Budget Summary</CardTitle>
    <CardDescription>Overview of project spending</CardDescription>
  </CardHeader>
  <CardContent>
    {/* Content */}
  </CardContent>
  <CardFooter>
    <Button variant="outline">View Details</Button>
  </CardFooter>
</Card>
```

---

## Dark Mode

### Implementation

```typescript
// ThemeProvider wraps app
import { ThemeProvider } from 'next-themes';

function App({ children }) {
  return (
    <ThemeProvider attribute="class" defaultTheme="system" enableSystem>
      {children}
    </ThemeProvider>
  );
}

// Theme toggle component
function ThemeToggle() {
  const { theme, setTheme } = useTheme();

  return (
    <Button
      variant="ghost"
      size="icon"
      onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}
    >
      <SunIcon className="h-4 w-4 rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
      <MoonIcon className="absolute h-4 w-4 rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
    </Button>
  );
}
```

---

## Responsive Breakpoints

| Breakpoint | Width | Usage |
|------------|-------|-------|
| `sm` | 640px | Mobile landscape |
| `md` | 768px | Tablet |
| `lg` | 1024px | Desktop |
| `xl` | 1280px | Large desktop |
| `2xl` | 1536px | Extra large |

### Common Patterns

```typescript
// Responsive grid
className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4"

// Responsive padding
className="p-4 sm:p-6 lg:p-8"

// Hide on mobile
className="hidden md:block"

// Show only on mobile
className="md:hidden"

// Responsive text
className="text-sm md:text-base"

// Responsive flex direction
className="flex flex-col md:flex-row"
```

---

## Animation

### Standard Transitions

```typescript
// Hover transitions
className="transition-colors hover:bg-accent"

// Scale on hover
className="transition-transform hover:scale-105"

// Fade in
className="animate-in fade-in duration-300"

// Slide in from bottom
className="animate-in slide-in-from-bottom duration-300"
```

### Loading Spinner

```typescript
import { Loader2 } from 'lucide-react';

<Loader2 className="h-4 w-4 animate-spin" />
```

# Phase 1: UI Pages

Authentication pages and initial frontend setup.

---

## Auth Layout

Shared layout for all auth pages.

```
+------------------------------------------------------------------+
|                                                                  |
|                         [HarvestIQ Logo]                         |
|                                                                  |
|                    +-------------------------+                   |
|                    |                         |                   |
|                    |    [Auth Form Here]     |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
|                                                                  |
|                    © 2024 HarvestIQ                              |
|                                                                  |
+------------------------------------------------------------------+
```

### Implementation

```typescript
// src/app/(auth)/layout.tsx
export default function AuthLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="min-h-screen flex items-center justify-center bg-muted/50">
      <div className="w-full max-w-md p-6">
        <div className="text-center mb-8">
          <Logo className="h-10 w-auto mx-auto" />
        </div>
        <Card>
          <CardContent className="pt-6">
            {children}
          </CardContent>
        </Card>
        <p className="text-center text-sm text-muted-foreground mt-6">
          © 2024 HarvestIQ
        </p>
      </div>
    </div>
  );
}
```

---

## Login Page

Route: `/login`

```
+------------------------------------------------------------------+
|                                                                  |
|                         [HarvestIQ Logo]                         |
|                                                                  |
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | Welcome back            |                   |
|                    | Sign in to your account |                   |
|                    |                         |                   |
|                    | Email                   |                   |
|                    | [_____________________] |                   |
|                    |                         |                   |
|                    | Password                |                   |
|                    | [_____________________] |                   |
|                    |                         |                   |
|                    | [Forgot password?]      |                   |
|                    |                         |                   |
|                    | [      Sign In      ]   |                   |
|                    |                         |                   |
|                    | Don't have an account?  |                   |
|                    | [Create one]            |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
|                                                                  |
+------------------------------------------------------------------+
```

### Form Fields

| Field | Type | Validation | Error Message |
|-------|------|------------|---------------|
| email | email | Required, valid email | Invalid email address |
| password | password | Required | Password is required |

### Actions

| Element | Action |
|---------|--------|
| Sign In button | Submit form, call POST /auth/login |
| Forgot password? | Navigate to /forgot-password |
| Create one | Navigate to /register |

### Error States

| Error Code | UI Response |
|------------|-------------|
| AUTH_1001 | "Invalid email or password" |
| AUTH_1007 | "Please verify your email first" + [Resend verification] link |
| AUTH_1008 | "Account locked. Try again in X minutes" |
| 429 | "Too many attempts. Please wait and try again" |

### States

| State | UI Change |
|-------|-----------|
| Loading | Button shows spinner, inputs disabled |
| Error | Error alert below form |
| Email Not Verified | Error with resend link |
| Account Locked | Error with remaining time |
| Success | Redirect to /dashboard |

---

## Register Page

Route: `/register`

```
+------------------------------------------------------------------+
|                                                                  |
|                         [HarvestIQ Logo]                         |
|                                                                  |
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | Create your account     |                   |
|                    |                         |                   |
|                    | Company Name            |                   |
|                    | [_____________________] |                   |
|                    |                         |                   |
|                    | Your Name               |                   |
|                    | [_____________________] |                   |
|                    |                         |                   |
|                    | Email                   |                   |
|                    | [_____________________] |                   |
|                    |                         |                   |
|                    | Password                |                   |
|                    | [_____________________] |                   |
|                    | Min 8 chars, upper,     |                   |
|                    | lower, number           |                   |
|                    |                         |                   |
|                    | Confirm Password        |                   |
|                    | [_____________________] |                   |
|                    |                         |                   |
|                    | [   Create Account   ]  |                   |
|                    |                         |                   |
|                    | Already have an account?|                   |
|                    | [Sign in]               |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
|                                                                  |
+------------------------------------------------------------------+
```

### Form Fields

| Field | Type | Validation | Error Message |
|-------|------|------------|---------------|
| company_name | text | Required, min 2 | Company name must be at least 2 characters |
| name | text | Required, min 2 | Name must be at least 2 characters |
| email | email | Required, valid email | Invalid email address |
| password | password | Required, min 8, upper, lower, number | Password requirements not met |
| confirmPassword | password | Must match password | Passwords do not match |

### Password Requirements Display

Show real-time validation:

```
Password requirements:
✓ At least 8 characters
✗ One uppercase letter
✓ One lowercase letter
✗ One number
```

### Success State (Check Email)

After successful registration, show this state instead of redirecting:

```
+------------------------------------------------------------------+
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | [✓] Check your email    |                   |
|                    |                         |                   |
|                    | We've sent a            |                   |
|                    | verification link to    |                   |
|                    | joey@example.com        |                   |
|                    |                         |                   |
|                    | Click the link in the   |                   |
|                    | email to verify your    |                   |
|                    | account.                |                   |
|                    |                         |                   |
|                    | Didn't receive it?      |                   |
|                    | [Resend email]          |                   |
|                    |                         |                   |
|                    | [Back to sign in]       |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
+------------------------------------------------------------------+
```

---

## Verify Email Page

Route: `/verify-email?token=xxx`

### Loading State

```
+------------------------------------------------------------------+
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | [Spinner]               |                   |
|                    |                         |                   |
|                    | Verifying your email... |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
+------------------------------------------------------------------+
```

### Success State

```
+------------------------------------------------------------------+
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | [✓] Email Verified      |                   |
|                    |                         |                   |
|                    | Your email has been     |                   |
|                    | verified successfully.  |                   |
|                    |                         |                   |
|                    | You can now sign in to  |                   |
|                    | your account.           |                   |
|                    |                         |                   |
|                    | [    Sign In    ]       |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
+------------------------------------------------------------------+
```

### Invalid/Expired Token State

```
+------------------------------------------------------------------+
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | [X] Verification Failed |                   |
|                    |                         |                   |
|                    | This verification link  |                   |
|                    | is invalid or has       |                   |
|                    | expired.                |                   |
|                    |                         |                   |
|                    | [Request new link]      |                   |
|                    |                         |                   |
|                    | [Back to sign in]       |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
+------------------------------------------------------------------+
```

### Implementation

```typescript
// src/app/(auth)/verify-email/page.tsx
'use client';

import { useEffect, useState } from 'react';
import { useSearchParams } from 'next/navigation';
import { api } from '@/lib/api';

export default function VerifyEmailPage() {
  const searchParams = useSearchParams();
  const token = searchParams.get('token');
  const [status, setStatus] = useState<'loading' | 'success' | 'error'>('loading');

  useEffect(() => {
    if (!token) {
      setStatus('error');
      return;
    }

    api.post('/auth/verify-email', { token })
      .then(() => setStatus('success'))
      .catch(() => setStatus('error'));
  }, [token]);

  if (status === 'loading') {
    return <VerifyingState />;
  }

  if (status === 'success') {
    return <SuccessState />;
  }

  return <ErrorState />;
}
```

---

## Forgot Password Page

Route: `/forgot-password`

```
+------------------------------------------------------------------+
|                                                                  |
|                         [HarvestIQ Logo]                         |
|                                                                  |
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | Reset your password     |                   |
|                    | Enter your email and    |                   |
|                    | we'll send a reset link |                   |
|                    |                         |                   |
|                    | Email                   |                   |
|                    | [_____________________] |                   |
|                    |                         |                   |
|                    | [   Send Reset Link  ]  |                   |
|                    |                         |                   |
|                    | [Back to sign in]       |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
|                                                                  |
+------------------------------------------------------------------+
```

### Success State

Always show success to prevent email enumeration:

```
+------------------------------------------------------------------+
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | Check your email        |                   |
|                    |                         |                   |
|                    | If an account exists    |                   |
|                    | for joey@example.com,   |                   |
|                    | we've sent a password   |                   |
|                    | reset link.             |                   |
|                    |                         |                   |
|                    | [Back to sign in]       |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
+------------------------------------------------------------------+
```

---

## Reset Password Page

Route: `/reset-password?token=xxx`

```
+------------------------------------------------------------------+
|                                                                  |
|                         [HarvestIQ Logo]                         |
|                                                                  |
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | Set new password        |                   |
|                    |                         |                   |
|                    | New Password            |                   |
|                    | [_____________________] |                   |
|                    |                         |                   |
|                    | Confirm Password        |                   |
|                    | [_____________________] |                   |
|                    |                         |                   |
|                    | [   Reset Password   ]  |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
|                                                                  |
+------------------------------------------------------------------+
```

### Success State

```
+------------------------------------------------------------------+
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | [✓] Password Reset      |                   |
|                    |                         |                   |
|                    | Your password has been  |                   |
|                    | reset successfully.     |                   |
|                    |                         |                   |
|                    | [    Sign In    ]       |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
+------------------------------------------------------------------+
```

### Invalid Token State

```
+------------------------------------------------------------------+
|                    +-------------------------+                   |
|                    |                         |                   |
|                    | [X] Invalid or Expired  |                   |
|                    |                         |                   |
|                    | This password reset     |                   |
|                    | link is no longer valid.|                   |
|                    |                         |                   |
|                    | [Request new link]      |                   |
|                    |                         |                   |
|                    +-------------------------+                   |
+------------------------------------------------------------------+
```

---

## Auth Store

```typescript
// src/stores/authStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface User {
  id: string;
  email: string;
  name: string;
  builderId: string;
}

interface AuthStore {
  user: User | null;
  setUser: (user: User | null) => void;
  clearUser: () => void;
}

export const useAuthStore = create<AuthStore>()(
  persist(
    (set) => ({
      user: null,
      setUser: (user) => set({ user }),
      clearUser: () => set({ user: null }),
    }),
    { name: 'auth-storage' }
  )
);
```

---

## Auth Hook

```typescript
// src/hooks/useAuth.ts
import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { useAuthStore } from '@/stores/authStore';
import { api } from '@/lib/api';

interface LoginInput {
  email: string;
  password: string;
}

interface RegisterInput {
  company_name: string;
  name: string;
  email: string;
  password: string;
}

export function useAuth() {
  const router = useRouter();
  const { user, setUser, clearUser } = useAuthStore();
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const login = async (data: LoginInput) => {
    setIsLoading(true);
    setError(null);
    try {
      const response = await api.post('/auth/login', data);
      setUser(response.data.user);
      router.push('/dashboard');
    } catch (err: any) {
      const errorCode = err.response?.data?.error?.code;

      if (errorCode === 'AUTH_1007') {
        setError('email_not_verified');
      } else if (errorCode === 'AUTH_1008') {
        setError('account_locked');
      } else {
        setError(err.response?.data?.error?.message || 'Login failed');
      }
      throw err;
    } finally {
      setIsLoading(false);
    }
  };

  const register = async (data: RegisterInput) => {
    setIsLoading(true);
    setError(null);
    try {
      await api.post('/auth/register', data);
      // Registration successful - don't set user, show check email message
      return { success: true, email: data.email };
    } catch (err: any) {
      setError(err.response?.data?.error?.message || 'Registration failed');
      throw err;
    } finally {
      setIsLoading(false);
    }
  };

  const logout = async () => {
    try {
      await api.post('/auth/logout');
    } finally {
      clearUser();
      router.push('/login');
    }
  };

  const refreshUser = async () => {
    try {
      const response = await api.get('/users/me');
      setUser(response.data);
    } catch {
      clearUser();
    }
  };

  const resendVerification = async (email: string) => {
    await api.post('/auth/resend-verification', { email });
  };

  return {
    user,
    isLoading,
    error,
    isAuthenticated: !!user,
    login,
    register,
    logout,
    refreshUser,
    resendVerification,
  };
}
```

---

## Protected Route Middleware

```typescript
// src/middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

const publicPaths = [
  '/login',
  '/register',
  '/verify-email',
  '/forgot-password',
  '/reset-password',
];

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  const token = request.cookies.get('access_token');

  // Allow public paths
  if (publicPaths.some(path => pathname.startsWith(path))) {
    // Redirect to dashboard if already logged in (except verify-email)
    if (token && !pathname.startsWith('/verify-email')) {
      return NextResponse.redirect(new URL('/dashboard', request.url));
    }
    return NextResponse.next();
  }

  // Protect all other paths
  if (!token) {
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('redirect', pathname);
    return NextResponse.redirect(loginUrl);
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

---

## API Client

```typescript
// src/lib/api.ts
const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL;

class ApiClient {
  private async request<T>(
    method: string,
    path: string,
    data?: any
  ): Promise<{ data: T }> {
    const response = await fetch(`${API_BASE_URL}${path}`, {
      method,
      headers: {
        'Content-Type': 'application/json',
      },
      credentials: 'include', // Include cookies
      body: data ? JSON.stringify(data) : undefined,
    });

    if (response.status === 401) {
      // Try to refresh token
      const refreshed = await this.refreshToken();
      if (refreshed) {
        // Retry original request
        return this.request(method, path, data);
      }
      // Redirect to login
      window.location.href = '/login';
      throw new Error('Session expired');
    }

    if (response.status === 429) {
      const error = await response.json();
      throw {
        response: {
          data: {
            error: {
              code: 'RATE_LIMITED',
              message: 'Too many requests. Please wait and try again.',
            },
          },
        },
      };
    }

    if (!response.ok) {
      const error = await response.json();
      throw { response: { data: error } };
    }

    const result = await response.json();
    return { data: result.data };
  }

  private async refreshToken(): Promise<boolean> {
    try {
      await fetch(`${API_BASE_URL}/auth/refresh`, {
        method: 'POST',
        credentials: 'include',
      });
      return true;
    } catch {
      return false;
    }
  }

  get<T>(path: string) {
    return this.request<T>('GET', path);
  }

  post<T>(path: string, data?: any) {
    return this.request<T>('POST', path, data);
  }

  put<T>(path: string, data?: any) {
    return this.request<T>('PUT', path, data);
  }

  delete<T>(path: string) {
    return this.request<T>('DELETE', path);
  }
}

export const api = new ApiClient();
```

# API Layer

## Three-Layer Pattern

1. **Service layer** (`src/services/<domain>/`): Raw Axios calls returning `AxiosResponse<T>`
2. **API hooks layer** (`src/api/<resource>.ts`): React Query hooks wrapping service calls
3. **Component layer**: Consumes API hooks — never calls Axios directly

## Single API Client Instance

Configure once, reuse everywhere:

```typescript
// src/lib/api-client.ts
import axios from "axios";

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  headers: {
    "Content-Type": "application/json",
  },
});

// Add interceptors for auth, error handling
apiClient.interceptors.request.use((config) => {
  const token = getAuthToken();
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Handle unauthorized - logout, refresh token, etc.
    }
    return Promise.reject(error);
  }
);
```

## API Request Structure

Split into service functions (raw calls) and API hooks (React Query wrappers):

### Service Layer

```typescript
// src/services/userService/userService.ts
import { apiClient } from "#/lib/api-client";
import type { User, CreateUserRequest } from "./types";

export const getUser = (userId: string) =>
  apiClient.get<User>(`/users/${userId}`);

export const getUsers = (params: { page?: number; size?: number } = {}) =>
  apiClient.get<Page<User>>("/users", { params });

export const createUser = (request: CreateUserRequest) =>
  apiClient.post<User>("/users", request);
```

### Types (co-located with service)

```typescript
// src/services/userService/types.ts
import { z } from "zod";

export const UserSchema = z.object({
  id: z.string(),
  email: z.string().email(),
  name: z.string(),
  role: z.enum(["admin", "user"]),
});

export type User = z.infer<typeof UserSchema>;

export interface CreateUserRequest {
  email: string;
  name: string;
  role: "admin" | "user";
}
```

### API Hooks Layer

```typescript
// src/api/users.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { getUser, getUsers, createUser } from "#/services/userService/userService";

export const useUser = (userId: string) =>
  useQuery({
    queryKey: ["users", userId],
    queryFn: () => getUser(userId).then((res) => res.data),
  });

export const useUsers = (params: { page?: number; size?: number } = {}) =>
  useQuery({
    queryKey: ["users", params],
    queryFn: () => getUsers(params).then((res) => res.data),
  });

export const useCreateUser = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (request: CreateUserRequest) =>
      createUser(request).then((res) => res.data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["users"] });
    },
  });
};
```

## Benefits of This Pattern

- **Separation of concerns** - Raw API calls vs. React integration
- **Discoverability** - Services in one place, hooks in another
- **Type safety** - Response typing flows through the app
- **Testability** - Easy to mock at the service or hook level

## Error Handling

### API Error Interceptors

Handle errors globally at the API client level:

```typescript
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    const message = error.response?.data?.message || "An error occurred";

    // Show toast notification
    toast.error(message);

    // Handle specific status codes
    if (error.response?.status === 401) {
      // Logout user or refresh token
      authStore.logout();
    }

    if (error.response?.status === 403) {
      // Redirect to unauthorized page
      router.navigate("/unauthorized");
    }

    return Promise.reject(error);
  }
);
```

### Error Boundaries

Use multiple error boundaries, not just one at the root:

```typescript
// ✅ Good - granular error boundaries
const App = () => (
  <RootErrorBoundary>
    <Layout>
      <Sidebar />
      <Main>
        <ErrorBoundary fallback={<DashboardError />}>
          <Dashboard />
        </ErrorBoundary>
      </Main>
    </Layout>
  </RootErrorBoundary>
);

// If Dashboard crashes, Sidebar stays functional
```

```typescript
// Error boundary component
import { Component, ReactNode } from "react";

type Props = {
  children: ReactNode;
  fallback: ReactNode;
};

type State = {
  hasError: boolean;
};

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(): State {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log to error tracking service (e.g., Sentry)
    console.error("Error caught by boundary:", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}
```

## Security

### Authentication Token Storage

| Method          | Security | Persistence     | XSS Risk   |
| --------------- | -------- | --------------- | ---------- |
| Memory (state)  | Highest  | Lost on refresh | None       |
| HttpOnly Cookie | High     | Persistent      | None       |
| localStorage    | Low      | Persistent      | Vulnerable |

**Recommendation:** Use HttpOnly cookies when possible. If using localStorage, ensure robust XSS prevention.

### Input Sanitization

Always sanitize user inputs before rendering:

```typescript
// ❌ Dangerous - XSS vulnerability
const Comment = ({ content }: { content: string }) => (
  <div dangerouslySetInnerHTML={{ __html: content }} />
);

// ✅ Safe - escaped by default
const Comment = ({ content }: { content: string }) => <div>{content}</div>;

// ✅ If HTML needed, sanitize first
import DOMPurify from "dompurify";

const Comment = ({ content }: { content: string }) => (
  <div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(content) }} />
);
```

### Authorization Patterns

**Role-Based Access Control (RBAC):**

```typescript
type Role = "admin" | "editor" | "viewer";

const PERMISSIONS = {
  admin: ["read", "write", "delete", "manage-users"],
  editor: ["read", "write"],
  viewer: ["read"],
} as const;

const hasPermission = (role: Role, permission: string): boolean => {
  return PERMISSIONS[role].includes(permission as any);
};

// Usage in components
const DeleteButton = () => {
  const { user } = useAuth();

  if (!hasPermission(user.role, "delete")) {
    return null;
  }

  return <button>Delete</button>;
};
```

**Permission-Based Access Control (PBAC):**

```typescript
// More granular - check specific resource ownership
const canDeleteComment = (user: User, comment: Comment): boolean => {
  return user.id === comment.authorId || user.role === "admin";
};
```

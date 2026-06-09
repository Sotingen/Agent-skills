# Performance

## Code Splitting

Split at the route level:

```typescript
import { lazy, Suspense } from "react";

const Dashboard = lazy(() => import("#/features/dashboard"));
const Settings = lazy(() => import("#/features/settings"));

const Router = () => (
  <Suspense fallback={<LoadingSpinner />}>
    <Routes>
      <Route path="/dashboard" element={<Dashboard />} />
      <Route path="/settings" element={<Settings />} />
    </Routes>
  </Suspense>
);
```

## State Optimization

**Split state by concern using separate Contexts:**

```typescript
// ❌ Bad - one massive context causes unnecessary re-renders
const AppContext = createContext<{
  user: User | null
  theme: string
  notifications: Notification[]
  // Everything re-renders when any value changes
}>(null!)

// ✅ Good - separate contexts per concern
const UserContext = createContext<UserContextType>(null!)
const ThemeContext = createContext<ThemeContextType>(null!)
const NotificationContext = createContext<NotificationContextType>(null!)
```

**Lazy state initialization:**

```typescript
// ❌ Bad - runs on every render
const [data, setData] = useState(expensiveComputation());

// ✅ Good - runs only once
const [data, setData] = useState(() => expensiveComputation());
```

## Children Optimization

Leverage children to prevent re-renders:

```typescript
// ❌ Bad - ExpensiveComponent re-renders when count changes
const Parent = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>{count}</button>
      <ExpensiveComponent />
    </div>
  );
};

// ✅ Good - ExpensiveComponent doesn't re-render
const Counter = ({ children }: { children: ReactNode }) => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>{count}</button>
      {children}
    </div>
  );
};

const Parent = () => (
  <Counter>
    <ExpensiveComponent />
  </Counter>
);
```

## Styling Performance

Use MUI's `sx` prop for styling. For frequently updating components, avoid inline object creation:

```typescript
// ❌ Bad - creates new object on every render
const FastUpdating = ({ value }: Props) => (
  <Box sx={{ display: 'flex', gap: 2 }}>{value}</Box>
)

// ✅ Good - stable reference
const containerSx = { display: 'flex', gap: 2 } as const

const FastUpdating = ({ value }: Props) => (
  <Box sx={containerSx}>{value}</Box>
)
```

For highly dynamic styles, use CSS variables via theme tokens.

## Image Optimization

```typescript
// Lazy loading
<img src={url} loading="lazy" alt="Description" />

// Responsive images
<img
  src={url}
  srcSet={`${smallUrl} 480w, ${mediumUrl} 800w, ${largeUrl} 1200w`}
  sizes="(max-width: 600px) 480px, (max-width: 900px) 800px, 1200px"
  alt="Description"
/>
```

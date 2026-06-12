# React TypeScript Type Safety

## Overview

TypeScript prevents runtime errors by catching type mismatches at compile time. This guide covers patterns for typing React components, props, hooks, and API boundaries correctly.

**Golden Rule**: Every component boundary (props in, callbacks out) should have an explicit type. Interior logic can rely on inference.

## Props — Always Use Interfaces

```tsx
// ❌ BAD: inline or untyped props
const MyComponent = ({ title, onClose }: any) => { ... }

// ❌ BAD: loose typing
const MyComponent = (props: Record<string, unknown>) => { ... }

// ✅ GOOD: explicit interface
interface MyComponentProps {
  title: string
  onClose: () => void
}

const MyComponent = ({ title, onClose }: MyComponentProps) => { ... }
```

## Props — Optional vs Required

Mark optional props with `?`. Never default required props to guard against `undefined`.

```tsx
interface UserCardProps {
  user: User                   // required — always present
  onEdit: (id: string) => void // required callback
  isSelected?: boolean          // optional — defaults to false via destructuring
  className?: string            // optional styling
}

const UserCard = ({
  user,
  onEdit,
  isSelected = false,
  className
}: UserCardProps) => { ... }
```

## Children Props

```tsx
// Use PropsWithChildren when the component accepts children
import { type PropsWithChildren } from 'react'

interface PanelProps {
  title: string
}

const Panel = ({ title, children }: PropsWithChildren<PanelProps>) => (
  <div>
    <h2>{title}</h2>
    {children}
  </div>
)
```

## Generic Components

Use generics for components that work with multiple data types (e.g., autocompletes, lists, tables).

```tsx
interface SelectFieldProps<T> {
  options: T[]
  value: T | null
  onChange: (value: T | null) => void
  getOptionLabel: (option: T) => string
  isOptionEqualToValue?: (option: T, value: T) => boolean
}

const SelectField = <T,>({
  options,
  value,
  onChange,
  getOptionLabel,
  isOptionEqualToValue
}: SelectFieldProps<T>) => { ... }
```

### Shared Comparators for Generic Components

When multiple components compare by the same field, extract a typed helper:

```tsx
const idEquals = <T extends { id: string }>(a: T, b: T) => a.id === b.id
```

## Event Handler Types

```tsx
// ❌ BAD: any or missing types
const handleChange = (e: any) => { ... }

// ✅ GOOD: use React event types
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  setValue(e.target.value)
}

// ✅ GOOD: for custom callbacks, define the signature in the props interface
interface FormProps {
  onSubmit: (data: UserForm) => Promise<void>
}
```

## Form Types with react-hook-form

```tsx
// Define the form shape as an interface
interface UserForm {
  name: string
  email: string
  role: Role | null
  department: Department | null
  startDate: Dayjs
  endDate: Dayjs | null
}

// useForm infers types from defaultValues, but explicit generic is safer
const { control, handleSubmit } = useForm<UserForm>({
  defaultValues: { ... }
})
```

## API Types — Request / Response Separation

```tsx
// Response type — what the API returns (read)
interface UserResponse {
  id: string
  name: string
  email: string
  role: string
  createdAt: string
}

// Request type — what we send (write) — often different from response
interface CreateUserRequest {
  name: string
  email: string
  role: string
}

// Keep them separate — never reuse the response type for requests
```

## Discriminated Unions

Use discriminated unions for state that has distinct modes:

```tsx
type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error }

// TypeScript narrows automatically in switch/if
const renderState = (state: AsyncState<User[]>) => {
  if (state.status === 'success') {
    return state.data.map(u => <Card key={u.id} user={u} />)
    //          ^^^^ TypeScript knows data exists here
  }
}
```

## Avoid These

| Anti-Pattern | Fix |
|---|---|
| `as any` | Add proper types or use `unknown` with type guards |
| `// @ts-ignore` | Fix the type error; it's warning you about a real problem |
| `!` (non-null assertion) | Use optional chaining `?.` or nullish coalescing `??` |
| Enum for simple string unions | Use `type Status = 'active' \| 'inactive'` |
| `Function` type | Use specific signature: `() => void` or `(id: string) => Promise<void>` |

## Checklist

1. Every component has a named props interface
2. Callbacks in props have explicit parameter and return types
3. API response types are separate from request types
4. Generic components use type parameters, not `any`
5. No `as any`, `@ts-ignore`, or untyped event handlers
6. Optional props use `?`, required props don't have fallback defaults

## Sources

- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)

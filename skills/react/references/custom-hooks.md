# React Custom Hooks — Separation of Concerns

## Overview

Custom hooks extract **non-UI logic** out of components, keeping components focused on rendering. This follows the Facade pattern: the component sees a simple interface while the hook encapsulates complexity.

**Golden Rule**: A component should describe **what** to render. A hook should describe **how** data and behavior work.

## When to Extract a Custom Hook

| Signal | Action |
|---|---|
| Component has 3+ `useState` calls that work together | Group into a hook |
| Component mixes API calls with rendering logic | Extract data layer into a hook |
| Same stateful logic is used in 2+ components | Share via a hook |
| Component file exceeds ~150 lines of logic (excluding JSX) | Split logic into hooks |
| `useEffect` + `useState` form a reusable pattern | Wrap in a hook |

## Pattern: Data Fetching + Mutations

```tsx
// ❌ BAD: data logic mixed into component
const ProductList = () => {
  const { data } = useQuery({ queryKey: ['products'], queryFn: fetchProducts })
  const mutation = useMutation({ mutationFn: createProduct })
  const [filter, setFilter] = useState('')
  const filtered = data?.filter(p => p.name.includes(filter)) ?? []
  const handleCreate = async (form: ProductForm) => {
    await mutation.mutateAsync(form)
    showNotification('Created')
  }
  return <div>...</div>
}

// ✅ GOOD: extract into a hook
const useProducts = (filter: string) => {
  const { data = [] } = useQuery({ queryKey: ['products'], queryFn: fetchProducts })
  const filtered = data.filter(p => p.name.includes(filter))
  return { products: filtered }
}

const useCreateProduct = () => {
  const mutation = useMutation({ mutationFn: createProduct })
  const handleCreate = async (form: ProductForm) => {
    await mutation.mutateAsync(form)
    showNotification('Created')
  }
  return { create: handleCreate, isPending: mutation.isPending }
}

const ProductList = () => {
  const [filter, setFilter] = useState('')
  const { products } = useProducts(filter)
  const { create, isPending } = useCreateProduct()
  return <div>...</div>
}
```

## Pattern: Form Logic

```tsx
// ✅ GOOD: form setup in a dedicated hook
const useProductForm = (defaults?: Partial<ProductForm>) => {
  return useForm<ProductForm>({
    defaultValues: {
      name: '',
      sku: '',
      description: '',
      ...defaults
    },
    mode: 'onChange',
    reValidateMode: 'onBlur'
  })
}
```

## Pattern: Feature Flag / Permission Check

```tsx
// ✅ GOOD: encapsulate feature checks
const useFeatureFlags = () => {
  const { features } = useContext(AppContext)
  const hasFeature = (id: string) => features.some(f => f.id === id && f.enabled)
  return { hasFeature }
}
```

## Naming Conventions

| Pattern | Naming | Example |
|---|---|---|
| Data access | `use<Entity>` | `useProducts`, `useUsers` |
| Mutation wrapper | `useCreate<Entity>`, `useUpdate<Entity>` | `useCreateProduct` |
| Form setup | `use<Entity>Form` | `useProductForm` |
| Feature check | `use<Feature>` | `useFeatureFlags`, `usePermissions` |
| UI behavior | `use<Behavior>` | `useSelection`, `useDrawer` |

## Hook Structure

```tsx
// 1. Import dependencies
import { useState, useCallback } from 'react'

// 2. Define return type (optional but recommended for complex hooks)
interface UseProductFormReturn {
  control: Control<ProductForm>
  handleSubmit: UseFormHandleSubmit<ProductForm>
  isValid: boolean
}

// 3. Hook function — prefix with "use"
export const useProductForm = (): UseProductFormReturn => {
  // 4. Call other hooks at the top
  const form = useForm<ProductForm>({ ... })

  // 5. Derive values
  const isValid = form.formState.isValid

  // 6. Define callbacks with useCallback if passed to children
  const handleSubmit = useCallback(() => { ... }, [])

  // 7. Return a clean interface
  return { control: form.control, handleSubmit, isValid }
}
```

## Rules

1. **Never call hooks conditionally** — hooks must be called in the same order every render
2. **Keep hooks pure** — no direct DOM manipulation; that belongs in `useEffect` inside the hook
3. **Don't over-abstract** — if logic is used once and is simple, keep it in the component
4. **Hooks can call other hooks** — compose small hooks into larger ones
5. **Co-locate hooks** near their consumers — put in same feature folder, not a global `hooks/` unless truly shared

## File Organization

```
features/
  products/
    hooks/
      useProductForm.ts       ← form hook for this feature
      usePriceCalculation.ts  ← feature-specific derived data
    pages/
      CreateProduct.tsx        ← component consumes hooks
```

## Sources

- [React Docs: Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)
- [React Docs: Extracting State Logic into a Reducer](https://react.dev/learn/extracting-state-logic-into-a-reducer)

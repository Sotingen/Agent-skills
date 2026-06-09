# React SPA Architecture Patterns

> Portable conventions for a React 19 + TypeScript + Vite single-page application.
> Use this guide when writing, reviewing, or refactoring code in the frontend app.

---

## Folder Structure

The app uses **hybrid organization**: layer-based at the root, feature-based inside `features/`.

```
src/
├── app/              # Application layer (routes, providers, router)
├── assets/           # Static files (images, fonts)
├── components/       # Shared components used across features
├── config/           # Global configuration and environment variables
├── features/         # Feature-based modules (primary organization)
├── hooks/            # Shared custom hooks
├── lib/              # Pre-configured library instances
├── stores/           # Global state stores
├── testing/          # Test utilities and mocks
├── types/            # Shared TypeScript type definitions
└── utils/            # Shared utility functions
```

### Feature Module Structure

Each feature is a self-contained module with its own internal structure:

```
src/features/<feature>/
├── api/              # API requests and data fetching hooks
├── components/       # Feature-specific components
├── hooks/            # Feature-specific custom hooks
├── stores/           # Feature state management
├── types/            # Feature TypeScript types
├── utils/            # Feature-specific utilities
└── index.ts          # Public API (what this feature exports)
```

**Only include folders that the feature needs.** A simple feature might only have `components/` and `api/`.

### Import Architecture

Enforce unidirectional code flow: **shared → features → app**

```
┌─────────────────────────────────────────────┐
│                    app/                      │
│         (composes features + shared)         │
└─────────────────────────────────────────────┘
                      ↑
┌─────────────────────────────────────────────┐
│                 features/                    │
│        (import from shared only)             │
│      ❌ Cannot import from other features    │
└─────────────────────────────────────────────┘
                      ↑
┌─────────────────────────────────────────────┐
│     shared (components, hooks, utils)        │
│           (no feature imports)               │
└─────────────────────────────────────────────┘
```

**Key rule:** Features cannot import from other features. Compose features at the app level instead.

---

## Data Fetching

### Three-layer pattern

1. **Service layer** (`src/services/<domain>/`): Raw Axios calls returning `AxiosResponse<T>`.
2. **API hooks layer** (`src/api/<resource>.ts`): React Query hooks wrapping service calls.
3. **Component layer**: Consumes API hooks — never calls Axios directly.

### Service functions

```typescript
// services/productService/productService.ts
export const pageProducts = (params: PageParams = {}) => axios.get<Page<Product>>("/api/products", { params });

export const getProduct = (id: string) => axios.get<Product>(`/api/products/${id}`);
```

- One file per domain entity
- Types co-located in a sibling `types.ts`
- Functions return raw `AxiosResponse` — let the caller unwrap `.data`

### React Query hooks

```typescript
// api/products.ts
export const useProductsQuery = (params: PageParams = {}) =>
	useQuery({
		queryKey: ["products", params],
		queryFn: () => pageProducts(params).then((res) => res.data),
	});

export const useCreateProductMutation = () => {
	const queryClient = useQueryClient();
	return useMutation({
		mutationFn: (request: CreateProductRequest) => createProduct(request).then((res) => res.data),
		onSuccess: (data) => {
			queryClient.invalidateQueries({ queryKey: ["products"] });
			queryClient.setQueryData(["product", data.id], data);
		},
	});
};
```

**Conventions:**

- Query keys: `['resource']` or `['resource', params]`
- Mutations invalidate related queries on success
- Optimistic updates via `setQueryData` where appropriate
- Default `staleTime: 60_000`, `retry: 5`, `refetchOnWindowFocus: false`

---

## Forms

### Library: react-hook-form

**Setup pattern** — extract form config into a custom hook:

```typescript
// hooks/useProductForm.ts
export const useProductForm = () =>
	useForm<ProductForm>({
		mode: "onChange",
		reValidateMode: "onBlur",
		defaultValues: {
			name: "",
			sku: "",
			description: "",
			price: 0,
			category: null,
		},
	});
```

### Controlled form components

Wrap MUI inputs with `useController` for reuse:

```typescript
const ControlledTextField = <T extends FieldValues>(props: ControlledTextFieldProps<T>) => {
    const { field, fieldState: { error } } = useController(props)
    return (
        <FormControl sx={props.sx}>
            <TextField
                {...field}
                label={props.label}
                error={!!error}
                helperText={error?.message}
                required={!!props.rules?.required}
            />
        </FormControl>
    )
}
```

- One `Controlled*` wrapper per MUI input type (TextField, Autocomplete, DatePicker, etc.)
- Located in `components/ControlledFormElements/`
- Generic over the form type: `<T extends FieldValues>`

### Edit form hooks

Complex edit forms get a dedicated hook that orchestrates fetching, resetting, and submitting:

```typescript
export const useEditProductForm = (productId: string) => {
	const form = useProductForm();
	const { data: product } = useProductQuery(productId);
	const mutation = useUpdateProductMutation();

	// Reset form when data loads
	useEffect(() => {
		if (product) form.reset(mapProductToForm(product));
	}, [product]);

	const onSubmit = form.handleSubmit((data) => mutation.mutate({ id: productId, request: mapFormToRequest(data) }));

	return { form, onSubmit, isPending: mutation.isPending };
};
```

---

## Component Patterns

### Facade hooks (primary pattern)

Extract all business logic into a custom hook; the component only renders:

```typescript
// hooks/useEditableField.ts
export function useEditableField<T extends FieldValues>(
    name: Path<T>,
    control: Control<T>
): EditableFieldState {
    const [isEditing, setIsEditing] = useState(false)
    const value = useWatch({ name, control })
    return { isEditing, value, startEdit, endEdit }
}

// components/EditableTitle.tsx
export function EditableTitle<T extends FieldValues>(props: EditableTitleProps<T>) {
    const { isEditing, value, startEdit, endEdit } = useEditableField(props.name, props.control)
    return isEditing ? <Controller ... /> : <Typography onClick={startEdit}>{value}</Typography>
}
```

### No render props, no HOCs

Hooks replace both patterns. If you need to share logic, write a hook.

### URL-driven modals

Use `useSearchParams()` for form/dialog open state — this makes modals linkable and back-button-friendly:

```typescript
const [searchParams, setSearchParams] = useSearchParams();
const isOpen = searchParams.get("mode") === "create";
const handleOpen = () => setSearchParams({ mode: "create" });
const handleClose = () => setSearchParams({});
```

---

## State Management

**No Redux / Zustand.** State is split across:

| Layer             | Tool                      | Use for                                  |
| ----------------- | ------------------------- | ---------------------------------------- |
| Server state      | React Query               | API data, caching, sync                  |
| App-wide UI state | React Context             | User info, permissions, snackbars        |
| Feature state     | React Context (scoped)    | Selection, navigation within a feature   |
| Component state   | `useState` / `useReducer` | Ephemeral UI state                       |
| Form state        | react-hook-form           | Input values, validation, dirty tracking |

### Context provider pattern

```typescript
// Providers/AppSnackbarProvider/AppSnackbarProvider.tsx
const AppSnackbarContext = createContext<AppSnackbarContextType | undefined>(undefined)

export const AppSnackbarProvider = ({ children }: PropsWithChildren) => {
    const [queue, setQueue] = useState<SnackbarItem[]>([])
    const showSnackbar = useCallback((item: SnackbarItem) => {
        setQueue((prev) => [...prev, item])
    }, [])
    return (
        <AppSnackbarContext.Provider value={{ showSnackbar }}>
            {children}
        </AppSnackbarContext.Provider>
    )
}

export const useAppSnackbar = () => {
    const ctx = useContext(AppSnackbarContext)
    if (!ctx) throw new Error('useAppSnackbar must be used within AppSnackbarProvider')
    return ctx
}
```

---

## Routing

**Library:** React Router v7

```typescript
const router = createBrowserRouter([
    {
        path: '/dashboard',
        element: <Suspense fallback={<Loading />}><Dashboard /></Suspense>,
        children: [
            { path: 'overview', element: <Overview /> },
            // ...
        ]
    },
    { path: '*', element: <Navigate to="/dashboard" /> }
], { basename: '/app' })
```

- Code-splitting with `React.lazy()` + `<Suspense>`
- Nested routes for layout composition
- `basename` set for microfrontend deployment

---

## Styling

**MUI v7 + sx prop** (primary approach):

```typescript
<Box sx={{ display: 'flex', flexDirection: 'column', gap: 2, p: 3 }}>
    <Typography variant="h6">{title}</Typography>
</Box>
```

- Use `sx` prop for one-off styles
- Use theme tokens via `useTheme()` for colors, spacing, breakpoints
- Avoid inline `style={}` — always prefer `sx`
- No CSS modules, no Tailwind

---

## Error Handling

### Three layers

1. **Axios interceptor** — catches all API errors, shows toast for unhandled ones:

```typescript
axios.interceptors.response.use(
	(response) => response,
	(error: AxiosError<ApiErrorDetail>) => {
		if (!error.config?.headers?.["x-skip-error-toast"]) {
			const message = getErrorMessage(error.response?.data);
			if (message) showSnackbar({ message, alert: { severity: "error" } });
		}
		return Promise.reject(error);
	},
);
```

2. **Component-level** — mutation `onError` callbacks for specific handling
3. **Error boundary** — catches render crashes with a fallback UI

### Structured API errors

```typescript
type ApiErrorDetail = ValidationErrorDetail | InvariantErrorDetail | NotFoundErrorDetail | ConflictErrorDetail;
// ...

function getErrorMessage(detail: ApiErrorDetail): string {
	return detail.detail || detail.title || "An unexpected error occurred";
}
```

Use `x-skip-error-toast` header to suppress the interceptor when handling errors locally.

---

## Testing

**Stack:** Vitest + React Testing Library + MSW (optional)

### Test file conventions

- Co-located: `Component.test.tsx` next to `Component.tsx`
- Or in the feature folder root for integration-style tests

### Render helper pattern

```typescript
const renderWithProviders = (ui: ReactElement) => {
    const queryClient = new QueryClient({
        defaultOptions: { queries: { retry: false } }
    })
    return render(
        <MemoryRouter>
            <QueryClientProvider client={queryClient}>
                {ui}
            </QueryClientProvider>
        </MemoryRouter>
    )
}
```

### Mock pattern

```typescript
vi.mock('#/api/products')
const mockUseProductsQuery = vi.mocked(useProductsQuery)

describe('ProductList', () => {
    it('renders empty state', () => {
        mockUseProductsQuery.mockReturnValue({ data: [], isPending: false } as any)
        renderWithProviders(<ProductList />)
        expect(screen.getByTestId('empty-state')).toBeInTheDocument()
    })
})
```

### Fixture factories

```typescript
// test-utils/fixtures.ts
export const makeProduct = (overrides: Partial<Product> = {}): Product => ({
	id: "prod-1",
	simpleId: "SKU-001",
	representation: { name: "Widget", price: 29.99 /* ... */ },
	...overrides,
});
```

### Global test setup

```typescript
// test.ts (setupFiles entry)
import "@testing-library/jest-dom/vitest";

vi.mock("react-i18next", () => ({
	useTranslation: () => ({
		t: (key: string) => key,
		i18n: { changeLanguage: () => Promise.resolve() },
	}),
}));
```

---

## Internationalization

**Library:** i18next + react-i18next

```typescript
const { t } = useTranslation()
<Button>{t('common.save', 'Save')}</Button>
```

- Keys use dot-notation namespacing: `'feature.section.label'`
- Always provide a fallback string as second argument
- Translation files live in `translations/<locale>/translation.json`
- Tests mock `useTranslation` globally — `t()` returns the key

---

## TypeScript Conventions

### Path alias

`#/` maps to `src/` — use it for all imports:

```typescript
import { useProductsQuery } from "#/api/products";
import { makeProduct } from "#/test-utils/fixtures";
```

### Type organization

- **Shared types** in `src/types/` (API error shapes, common models)
- **Domain types** co-located with services: `services/<domain>/types.ts`
- **Component props** defined inline or in the same file
- **Generic components** use constrained generics: `<T extends FieldValues>`

### Common generic patterns

```typescript
// Representation wrapper (API response shape)
type Representation<TRep, TLinks = {}> = {
	id: string;
	simpleId?: string;
	representation: TRep;
	_links?: TLinks;
};

// Page response
type Page<T> = {
	items: T[];
	page: number;
	size: number;
	totalElements: number;
	_links?: { next?: Link };
};
```

---

## Global Custom Hooks

| Hook                                  | Purpose                                              |
| ------------------------------------- | ---------------------------------------------------- |
| `useDebounce(value, delay)`           | Debounce a reactive value                            |
| `useDeleteHandler(mutation, options)` | Confirm + execute delete with error/success feedback |
| `useError()`                          | Show error snackbar via context                      |
| `useSuccessSnackbar()`                | Show success snackbar via context                    |

Place reusable hooks in `src/hooks/`. Feature-specific hooks go in `features/<feature>/hooks/`.

---

## Quick Reference

| Concern        | Pattern                                                               |
| -------------- | --------------------------------------------------------------------- |
| Fetch data     | Service function → React Query hook → component                       |
| Forms          | `useForm()` in hook → `Controlled*` components in JSX                 |
| Business logic | Custom hook (facade) → thin component                                 |
| Global state   | Context provider + `useX()` hook                                      |
| Modals/dialogs | URL search params (`useSearchParams`)                                 |
| Styling        | MUI `sx` prop + theme tokens                                          |
| Testing        | Vitest + RTL + vi.mock API hooks + fixture factories                  |
| Error handling | Axios interceptor (global) + onError (local) + ErrorBoundary (render) |
| i18n           | `t('dotted.key', 'Default text')`                                     |
| Imports        | `#/path` alias for `src/`                                             |

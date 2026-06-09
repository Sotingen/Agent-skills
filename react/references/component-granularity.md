# React Component Granularity

## Overview

Large components are hard to read, test, and reuse. This guide helps identify when and how to split components into smaller, focused units.

**Golden Rule**: A component should do **one thing**. If you need the word "and" to describe it, consider splitting.

## When to Split

| Signal | Likely Action |
|---|---|
| Component exceeds ~200 lines | Extract sections into child components |
| A block of JSX is repeated with slight variations | Extract a reusable component |
| A section has its own state that doesn't affect siblings | Extract into its own component |
| You're passing 8+ props to configure one section | That section is its own component |
| Conditional rendering produces two very different UIs | Split into separate components, select with a parent |
| A section heading + fields form a logical group | Extract as a form section component |

## When NOT to Split

- The component is already under ~100 lines and reads clearly
- Splitting would require prop-drilling through 3+ levels (consider context or composition instead)
- The "component" would have only 1 consumer and no reuse potential — a local `const` JSX variable may suffice

## Pattern 1: Extract Repeated UI Blocks

```tsx
// ❌ BAD: repeated heading pattern
<Box>
  <Typography variant="h3">Capacity</Typography>
  <Tooltip title="..."><IconButton>?</IconButton></Tooltip>
</Box>
// ... (same pattern repeated for other sections)

// ✅ GOOD: extract into a component
const SectionHeading = ({ title, tooltip, ariaLabel }: SectionHeadingProps) => (
  <Box sx={{ display: 'flex', alignItems: 'center', gap: '6px' }}>
    <Typography variant="h3">{title}</Typography>
    <Tooltip title={tooltip}>
      <IconButton size="small" aria-label={ariaLabel}>
        <InfoIcon />
      </IconButton>
    </Tooltip>
  </Box>
)
```

## Pattern 2: Form Sections

Group related form fields into section components:

```tsx
// ❌ BAD: 300-line form component with all fields inline
const CreateUserForm = () => {
  return (
    <form>
      {/* 15 fields all inline */}
    </form>
  )
}

// ✅ GOOD: logical sections
const CreateUserForm = () => {
  const { control } = useUserForm()
  return (
    <form>
      <PersonalInfoFields control={control} />
      <AddressFields control={control} />
      {showExtendedFields && <ExtendedFields control={control} />}
      <FormActions onClose={onClose} isValid={isValid} />
    </form>
  )
}
```

## Pattern 3: Conditional Rendering → Separate Components

```tsx
// ❌ BAD: two completely different UIs in one component
const UserView = ({ mode }: { mode: 'view' | 'edit' }) => {
  if (mode === 'edit') {
    return (/* 100 lines of edit form */)
  }
  return (/* 100 lines of read-only view */)
}

// ✅ GOOD: separate components, parent selects
const UserView = ({ mode }: { mode: 'view' | 'edit' }) => {
  return mode === 'edit'
    ? <EditUser />
    : <ViewUser />
}
```

## Pattern 4: Inline JSX Variables (Lightweight Split)

When a section is complex but only used once and doesn't need its own state, extract to a local variable:

```tsx
const MyComponent = () => {
  const headerContent = (
    <Box sx={{ display: 'flex', justifyContent: 'space-between' }}>
      <Typography variant="h2">{title}</Typography>
      <Button onClick={onClose}>Close</Button>
    </Box>
  )

  return (
    <div>
      {headerContent}
      <MainContent />
    </div>
  )
}
```

## Component File Organization

```
features/
  users/
    components/
      UserFormSection.tsx        ← reusable form section
      EditableRow.tsx            ← generic editable row
    pages/
      CreateUser.tsx             ← page-level component
      EditUser.tsx               ← page-level component
```

- **Page/route components** in feature-specific folders
- **Shared UI blocks** in `components/` within the feature
- **Truly global components** in `src/components/`

## Props Interface Design

Keep props interfaces focused. If a component needs 10+ props, it may be doing too much:

```tsx
// ❌ Smell: too many props
interface MegaFormProps {
  name: string
  email: string
  phone: string
  address: Address
  role: Role
  department: Department
  manager: User
  startDate: Date
  salary: number
  notes: string
  onSubmit: () => void
  onClose: () => void
}

// ✅ Better: pass form control, let sections manage their own fields
interface AddressSectionProps<T extends FieldValues> {
  control: Control<T>
}
```

## Checklist

1. Can I describe this component's purpose in one short sentence?
2. Does any JSX block repeat with variations? → Extract component
3. Does a section have its own independent state? → Extract component
4. Is the props list growing past 6-8 items? → Consider splitting
5. Would extracting this section make the parent easier to test? → Do it

## Sources

- [React Docs: Thinking in React](https://react.dev/learn/thinking-in-react)
- [React Docs: Extracting Components](https://react.dev/learn/your-first-component)

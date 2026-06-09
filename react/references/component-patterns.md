# Component Patterns

## Colocation Principle

Keep components, functions, styles, and state as close as possible to where they're used:

```
// ✅ Good - colocated
src/features/checkout/
├── components/
│   ├── CheckoutForm.tsx
│   ├── CheckoutForm.test.tsx       # Test next to component
│   └── useCheckoutForm.ts          # Hook used only by this component

// ❌ Bad - scattered
src/
├── components/CheckoutForm.tsx
├── hooks/useCheckoutForm.ts        # Far from where it's used
├── tests/CheckoutForm.test.tsx     # Even further away
```

## No Nested Render Functions

Extract UI units into separate components:

```typescript
import { Avatar, Box, Typography } from '@mui/material'

// ❌ Bad - nested render function
const UserList = ({ users }: Props) => {
  const renderUser = (user: User) => (
    <Box sx={{ display: 'flex', alignItems: 'center', gap: 1, p: 2 }}>
      <Avatar src={user.avatar} />
      <Typography>{user.name}</Typography>
    </Box>
  );

  return <Box>{users.map(renderUser)}</Box>;
};

// ✅ Good - extracted component
const UserCard = ({ user }: { user: User }) => (
  <Box sx={{ display: 'flex', alignItems: 'center', gap: 1, p: 2 }}>
    <Avatar src={user.avatar} />
    <Typography>{user.name}</Typography>
  </Box>
);

const UserList = ({ users }: Props) => (
  <Box>
    {users.map((user) => (
      <UserCard key={user.id} user={user} />
    ))}
  </Box>
);
```

## Composition Over Props

Use children and slots instead of excessive props:

```typescript
import { Box, Typography, type SxProps, type Theme, type PropsWithChildren } from '@mui/material'

// ❌ Bad - prop drilling, inflexible
type CardProps = {
  title: string;
  subtitle: string;
  icon: ReactNode;
  actions: ReactNode;
  footer: ReactNode;
  // ... props keep growing
};

// ✅ Good - composition with MUI
interface SlotProps extends PropsWithChildren {
  sx?: SxProps<Theme>
}

const Card = ({ children, sx }: SlotProps) => (
  <Box sx={{ border: 1, borderColor: 'divider', borderRadius: 2, ...sx }}>
    {children}
  </Box>
);

const CardHeader = ({ children, sx }: SlotProps) => (
  <Box sx={{ p: 2, borderBottom: 1, borderColor: 'divider', ...sx }}>
    {children}
  </Box>
);

const CardBody = ({ children, sx }: SlotProps) => (
  <Box sx={{ p: 2, ...sx }}>{children}</Box>
);

// Usage - flexible composition
<Card>
  <CardHeader>
    <Icon name="user" />
    <Typography variant="h6">User Profile</Typography>
  </CardHeader>
  <CardBody>
    <UserDetails user={user} />
  </CardBody>
</Card>;
```

## Wrap Third-Party Components

Insulate your application from dependency changes:

```typescript
// ✅ Good - wrapped third-party component
// src/components/ui/date-picker.tsx
import { DatePicker as ThirdPartyDatePicker } from "some-library";

type DatePickerProps = {
  value: Date | null;
  onChange: (date: Date | null) => void;
  minDate?: Date;
  maxDate?: Date;
};

export const DatePicker = ({
  value,
  onChange,
  minDate,
  maxDate,
}: DatePickerProps) => {
  return (
    <ThirdPartyDatePicker
      selected={value}
      onSelect={onChange}
      minDate={minDate}
      maxDate={maxDate}
    />
  );
};

// If library changes, only update this file
```

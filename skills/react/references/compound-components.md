# Compound Components

Multiple related components that work together to form a cohesive UI pattern.

## Pattern Overview

Compound components provide:
- Flexible composition
- Shared context
- Consistent styling
- Semantic HTML structure

## Card Component Example

Using MUI's `sx` prop for styling:

```typescript
// components/ui/Card.tsx
import { Box, Typography, type SxProps, type Theme } from '@mui/material'
import { type PropsWithChildren } from 'react'

interface CardProps extends PropsWithChildren {
  sx?: SxProps<Theme>
}

export const Card = ({ children, sx }: CardProps) => (
  <Box
    sx={{
      borderRadius: 2,
      border: 1,
      borderColor: 'divider',
      bgcolor: 'background.paper',
      boxShadow: 1,
      ...sx,
    }}
  >
    {children}
  </Box>
)

export const CardHeader = ({ children, sx }: CardProps) => (
  <Box sx={{ display: 'flex', flexDirection: 'column', gap: 0.5, p: 3, ...sx }}>
    {children}
  </Box>
)

export const CardTitle = ({ children, sx }: CardProps) => (
  <Typography variant="h6" sx={{ fontWeight: 600, ...sx }}>
    {children}
  </Typography>
)

export const CardDescription = ({ children, sx }: CardProps) => (
  <Typography variant="body2" color="text.secondary" sx={sx}>
    {children}
  </Typography>
)

export const CardContent = ({ children, sx }: CardProps) => (
  <Box sx={{ px: 3, pb: 3, ...sx }}>{children}</Box>
)

export const CardFooter = ({ children, sx }: CardProps) => (
  <Box sx={{ display: 'flex', alignItems: 'center', px: 3, pb: 3, ...sx }}>
    {children}
  </Box>
)
```

## Usage Example

```tsx
import { Button } from '@mui/material'
import { Card, CardHeader, CardTitle, CardDescription, CardContent, CardFooter } from '#/components/ui/Card'

<Card>
  <CardHeader>
    <CardTitle>Account</CardTitle>
    <CardDescription>Manage your account settings</CardDescription>
  </CardHeader>
  <CardContent>
    <form>...</form>
  </CardContent>
  <CardFooter>
    <Button variant="contained">Save</Button>
  </CardFooter>
</Card>
```

## Design Principles

- Each component has a single responsibility
- Components are independently styleable
- Consistent spacing and typography
- Semantic HTML elements (div, h3, p)

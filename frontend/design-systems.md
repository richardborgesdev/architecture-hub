# Design Systems

## Introduction

A design system is a collection of reusable components, guidelines, and standards that help teams build consistent, accessible, and scalable user interfaces.

## What is a Design System?

A design system consists of:

1. **Design tokens** - Visual design atoms (colors, typography, spacing)
2. **Component library** - Reusable UI components
3. **Guidelines** - Usage patterns and best practices
4. **Documentation** - How to use the system
5. **Tooling** - Development and design tools

## Why Design Systems?

**Benefits:**
- **Consistency** - Unified look and feel
- **Efficiency** - Faster development
- **Scalability** - Easier to maintain large applications
- **Accessibility** - Built-in a11y standards
- **Collaboration** - Shared language between design and development

**Challenges:**
- Initial investment
- Maintenance overhead
- Adoption across teams
- Versioning and updates
- Balance between flexibility and consistency

## Architecture

### Atomic Design Methodology

Break interfaces into fundamental building blocks:

```
Atoms → Molecules → Organisms → Templates → Pages
```

#### 1. Atoms

Smallest building blocks (buttons, inputs, labels).

```tsx
// Button atom
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  children: ReactNode;
  onClick?: () => void;
}

export const Button = ({
  variant = 'primary',
  size = 'md',
  disabled = false,
  children,
  onClick,
}: ButtonProps) => {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
};

// Input atom
interface InputProps {
  type?: 'text' | 'email' | 'password';
  placeholder?: string;
  value: string;
  onChange: (value: string) => void;
  error?: string;
}

export const Input = ({
  type = 'text',
  placeholder,
  value,
  onChange,
  error,
}: InputProps) => {
  return (
    <div className="input-wrapper">
      <input
        type={type}
        placeholder={placeholder}
        value={value}
        onChange={(e) => onChange(e.target.value)}
        className={error ? 'input input-error' : 'input'}
      />
      {error && <span className="error-message">{error}</span>}
    </div>
  );
};
```

#### 2. Molecules

Simple groups of atoms (search bar, form field).

```tsx
// Form field molecule
interface FormFieldProps {
  label: string;
  error?: string;
  required?: boolean;
  children: ReactNode;
}

export const FormField = ({
  label,
  error,
  required,
  children,
}: FormFieldProps) => {
  return (
    <div className="form-field">
      <label className="form-label">
        {label}
        {required && <span className="required">*</span>}
      </label>
      {children}
      {error && <span className="error-message">{error}</span>}
    </div>
  );
};

// Search bar molecule
export const SearchBar = ({
  placeholder = 'Search...',
  onSearch,
}: {
  placeholder?: string;
  onSearch: (query: string) => void;
}) => {
  const [query, setQuery] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSearch(query);
  };

  return (
    <form onSubmit={handleSubmit} className="search-bar">
      <Input
        placeholder={placeholder}
        value={query}
        onChange={setQuery}
      />
      <Button type="submit" variant="primary">
        Search
      </Button>
    </form>
  );
};
```

#### 3. Organisms

Complex components (navigation bar, card grid).

```tsx
// Navigation organism
interface NavItem {
  label: string;
  href: string;
  icon?: ReactNode;
}

interface NavigationProps {
  logo: ReactNode;
  items: NavItem[];
  user?: User;
  onLogout?: () => void;
}

export const Navigation = ({
  logo,
  items,
  user,
  onLogout,
}: NavigationProps) => {
  return (
    <nav className="navigation">
      <div className="nav-logo">{logo}</div>

      <ul className="nav-items">
        {items.map((item) => (
          <li key={item.href}>
            <a href={item.href} className="nav-link">
              {item.icon && <span className="nav-icon">{item.icon}</span>}
              {item.label}
            </a>
          </li>
        ))}
      </ul>

      {user && (
        <div className="nav-user">
          <Avatar src={user.avatar} alt={user.name} />
          <Button variant="ghost" onClick={onLogout}>
            Logout
          </Button>
        </div>
      )}
    </nav>
  );
};
```

## Design Tokens

Define visual design decisions as data.

```typescript
// tokens/colors.ts
export const colors = {
  // Brand
  brand: {
    primary: '#0066CC',
    secondary: '#6C757D',
    accent: '#FF6B6B',
  },

  // Neutral
  neutral: {
    white: '#FFFFFF',
    black: '#000000',
    gray: {
      50: '#F8F9FA',
      100: '#E9ECEF',
      200: '#DEE2E6',
      300: '#CED4DA',
      400: '#ADB5BD',
      500: '#6C757D',
      600: '#495057',
      700: '#343A40',
      800: '#212529',
      900: '#0D1117',
    },
  },

  // Semantic
  semantic: {
    success: '#28A745',
    warning: '#FFC107',
    error: '#DC3545',
    info: '#17A2B8',
  },
} as const;

// tokens/typography.ts
export const typography = {
  fontFamily: {
    base: "'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif",
    mono: "'Fira Code', 'Courier New', monospace",
  },

  fontSize: {
    xs: '0.75rem',    // 12px
    sm: '0.875rem',   // 14px
    base: '1rem',     // 16px
    lg: '1.125rem',   // 18px
    xl: '1.25rem',    // 20px
    '2xl': '1.5rem',  // 24px
    '3xl': '1.875rem', // 30px
    '4xl': '2.25rem', // 36px
    '5xl': '3rem',    // 48px
  },

  fontWeight: {
    light: 300,
    normal: 400,
    medium: 500,
    semibold: 600,
    bold: 700,
  },

  lineHeight: {
    tight: 1.25,
    normal: 1.5,
    relaxed: 1.75,
  },
} as const;

// tokens/spacing.ts
export const spacing = {
  0: '0',
  1: '0.25rem',  // 4px
  2: '0.5rem',   // 8px
  3: '0.75rem',  // 12px
  4: '1rem',     // 16px
  5: '1.25rem',  // 20px
  6: '1.5rem',   // 24px
  8: '2rem',     // 32px
  10: '2.5rem',  // 40px
  12: '3rem',    // 48px
  16: '4rem',    // 64px
  20: '5rem',    // 80px
} as const;

// tokens/breakpoints.ts
export const breakpoints = {
  sm: '640px',
  md: '768px',
  lg: '1024px',
  xl: '1280px',
  '2xl': '1536px',
} as const;
```

### Using Design Tokens

```tsx
// Create theme object
import { colors, typography, spacing } from './tokens';

export const theme = {
  colors,
  typography,
  spacing,
} as const;

// Use in styled-components
import styled from 'styled-components';

const Button = styled.button`
  padding: ${({ theme }) => `${theme.spacing[2]} ${theme.spacing[4]}`};
  font-family: ${({ theme }) => theme.typography.fontFamily.base};
  font-size: ${({ theme }) => theme.typography.fontSize.base};
  background-color: ${({ theme }) => theme.colors.brand.primary};
  color: ${({ theme }) => theme.colors.neutral.white};
  border-radius: 4px;

  &:hover {
    background-color: ${({ theme }) => theme.colors.brand.secondary};
  }
`;

// Or use in CSS custom properties
:root {
  --color-primary: ${colors.brand.primary};
  --color-secondary: ${colors.brand.secondary};
  --spacing-2: ${spacing[2]};
  --spacing-4: ${spacing[4]};
}
```

## Component API Design

### Principles

1. **Consistent naming** - Use clear, predictable names
2. **Composable** - Components work together
3. **Flexible** - Support common use cases
4. **Type-safe** - TypeScript for better DX
5. **Accessible** - ARIA attributes by default

```tsx
// Good component API
interface ButtonProps {
  // Visual variants
  variant?: 'primary' | 'secondary' | 'ghost' | 'danger';
  size?: 'sm' | 'md' | 'lg';

  // States
  disabled?: boolean;
  loading?: boolean;

  // Content
  children: ReactNode;
  leftIcon?: ReactNode;
  rightIcon?: ReactNode;

  // Behavior
  onClick?: () => void;
  type?: 'button' | 'submit' | 'reset';

  // Styling
  className?: string;
  fullWidth?: boolean;
}

export const Button = ({
  variant = 'primary',
  size = 'md',
  disabled = false,
  loading = false,
  children,
  leftIcon,
  rightIcon,
  onClick,
  type = 'button',
  className,
  fullWidth = false,
}: ButtonProps) => {
  return (
    <button
      type={type}
      className={cn(
        'btn',
        `btn-${variant}`,
        `btn-${size}`,
        fullWidth && 'btn-full-width',
        className
      )}
      disabled={disabled || loading}
      onClick={onClick}
      aria-busy={loading}
    >
      {loading && <Spinner size="sm" />}
      {!loading && leftIcon && <span className="btn-icon-left">{leftIcon}</span>}
      <span className="btn-content">{children}</span>
      {!loading && rightIcon && <span className="btn-icon-right">{rightIcon}</span>}
    </button>
  );
};

// Usage examples
<Button>Default</Button>
<Button variant="secondary" size="lg">Large Secondary</Button>
<Button loading>Loading...</Button>
<Button leftIcon={<SaveIcon />}>Save</Button>
<Button fullWidth>Full Width</Button>
```

## Documentation

Use Storybook for component documentation:

```tsx
// Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'ghost', 'danger'],
      description: 'Visual style of the button',
    },
    size: {
      control: 'select',
      options: ['sm', 'md', 'lg'],
    },
    disabled: {
      control: 'boolean',
    },
  },
};

export default meta;
type Story = StoryObj<typeof Button>;

export const Primary: Story = {
  args: {
    children: 'Primary Button',
    variant: 'primary',
  },
};

export const Secondary: Story = {
  args: {
    children: 'Secondary Button',
    variant: 'secondary',
  },
};

export const WithIcons: Story = {
  args: {
    children: 'Save',
    leftIcon: <SaveIcon />,
  },
};

export const Loading: Story = {
  args: {
    children: 'Loading...',
    loading: true,
  },
};
```

## Accessibility

Build accessibility into components:

```tsx
// Accessible button
export const Button = (props: ButtonProps) => {
  const {
    children,
    disabled,
    loading,
    onClick,
    ariaLabel,
    ...rest
  } = props;

  return (
    <button
      {...rest}
      onClick={onClick}
      disabled={disabled || loading}
      aria-label={ariaLabel}
      aria-busy={loading}
      aria-disabled={disabled}
    >
      {children}
    </button>
  );
};

// Accessible modal
export const Modal = ({
  isOpen,
  onClose,
  title,
  children,
}: ModalProps) => {
  useEffect(() => {
    if (isOpen) {
      // Trap focus
      const previousFocus = document.activeElement;
      // Focus first focusable element

      return () => {
        // Restore focus
        (previousFocus as HTMLElement)?.focus();
      };
    }
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
    >
      <div className="modal-backdrop" onClick={onClose} />
      <div className="modal-content">
        <h2 id="modal-title">{title}</h2>
        <button
          onClick={onClose}
          aria-label="Close modal"
        >
          ×
        </button>
        {children}
      </div>
    </div>
  );
};
```

## Versioning & Distribution

### Package Structure

```
design-system/
├── packages/
│   ├── tokens/           # Design tokens
│   ├── components/       # React components
│   ├── icons/           # Icon library
│   └── utils/           # Utilities
├── docs/                # Documentation site
└── examples/            # Example apps
```

### Publishing

```json
// package.json
{
  "name": "@company/design-system",
  "version": "1.0.0",
  "main": "dist/index.js",
  "module": "dist/index.esm.js",
  "types": "dist/index.d.ts",
  "files": ["dist"],
  "peerDependencies": {
    "react": ">=18.0.0",
    "react-dom": ">=18.0.0"
  }
}
```

### Semantic Versioning

- **Major** (1.0.0): Breaking changes
- **Minor** (0.1.0): New features, backward compatible
- **Patch** (0.0.1): Bug fixes

## Testing

```tsx
// Component testing
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('renders children', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click</Button>);
    fireEvent.click(screen.getByText('Click'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('is disabled when loading', () => {
    render(<Button loading>Loading</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
});

// Visual regression testing with Chromatic
```

## Best Practices

### ✅ Do

- Start with design tokens
- Document everything
- Build for accessibility
- Version your components
- Provide usage examples
- Test thoroughly
- Listen to feedback
- Iterate based on usage

### ❌ Don't

- Build components you don't need
- Create overly complex APIs
- Ignore accessibility
- Skip documentation
- Make breaking changes without notice
- Couple to specific frameworks unnecessarily
- Forget about performance

## Further Reading

- [Component Architecture](./component-architecture.md)
- [Frontend Patterns](../patterns/frontend-patterns.md)
- [Accessibility Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)

---

*See also: [Micro-Frontends](./micro-frontends.md) | [Performance](./performance.md)*

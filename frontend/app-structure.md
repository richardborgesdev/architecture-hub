# Application Structure

## Introduction

How you structure your frontend application directly impacts maintainability, scalability, and team productivity. This guide covers proven approaches to organizing large-scale frontend applications.

## Structural Approaches

### 1. Feature-Based Structure (Recommended)

Organize by business features/domains:

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   ├── RegisterForm.tsx
│   │   │   └── PasswordReset.tsx
│   │   ├── hooks/
│   │   │   ├── useAuth.ts
│   │   │   └── useSession.ts
│   │   ├── services/
│   │   │   └── authService.ts
│   │   ├── store/
│   │   │   └── authSlice.ts
│   │   ├── types/
│   │   │   └── index.ts
│   │   └── index.ts
│   ├── dashboard/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── index.ts
│   ├── products/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── index.ts
│   └── cart/
│       ├── components/
│       ├── hooks/
│       ├── store/
│       └── index.ts
├── shared/
│   ├── components/
│   │   ├── Button/
│   │   ├── Input/
│   │   ├── Modal/
│   │   └── Table/
│   ├── hooks/
│   │   ├── useDebounce.ts
│   │   ├── useLocalStorage.ts
│   │   └── useMediaQuery.ts
│   ├── utils/
│   │   ├── format.ts
│   │   ├── validation.ts
│   │   └── date.ts
│   └── types/
│       └── common.ts
├── core/
│   ├── api/
│   │   ├── client.ts
│   │   ├── interceptors.ts
│   │   └── endpoints.ts
│   ├── config/
│   │   ├── env.ts
│   │   └── constants.ts
│   ├── routing/
│   │   ├── Router.tsx
│   │   ├── routes.ts
│   │   └── guards.ts
│   └── store/
│       ├── store.ts
│       └── rootReducer.ts
├── layouts/
│   ├── AppLayout.tsx
│   ├── AuthLayout.tsx
│   └── DashboardLayout.tsx
├── pages/
│   ├── Home.tsx
│   ├── Login.tsx
│   ├── Dashboard.tsx
│   └── NotFound.tsx
└── assets/
    ├── images/
    ├── fonts/
    └── styles/
```

**Advantages:**
- Features are self-contained
- Easy to find related code
- Scales with application complexity
- Team members can work independently
- Easy to extract features as packages

**When to use:**
- Medium to large applications
- Multiple developers
- Long-term projects
- Domain-driven design

### 2. Layer-Based Structure

Organize by technical concerns:

```
src/
├── components/
│   ├── auth/
│   ├── products/
│   └── shared/
├── hooks/
│   ├── useAuth.ts
│   ├── useProducts.ts
│   └── useCart.ts
├── services/
│   ├── authService.ts
│   ├── productService.ts
│   └── cartService.ts
├── store/
│   ├── slices/
│   └── store.ts
├── utils/
├── types/
└── pages/
```

**Advantages:**
- Simple to understand initially
- Clear separation of concerns
- Good for small applications

**Disadvantages:**
- Files related to same feature are scattered
- Harder to maintain as app grows
- Difficult to split into modules

**When to use:**
- Small applications
- Prototypes
- Solo developers
- Simple requirements

### 3. Hybrid Approach

Combine feature-based and layer-based:

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── index.ts
│   └── products/
│       ├── components/
│       ├── hooks/
│       └── index.ts
├── shared/
│   ├── components/
│   ├── hooks/
│   └── utils/
└── core/
    ├── api/
    ├── routing/
    └── store/
```

**When to use:**
- Most production applications
- Balance between organization and flexibility

## Detailed Structure Patterns

### Component Organization

```
components/
├── Button/
│   ├── Button.tsx          # Main component
│   ├── Button.test.tsx     # Tests
│   ├── Button.stories.tsx  # Storybook
│   ├── Button.styles.ts    # Styles
│   ├── Button.types.ts     # TypeScript types
│   └── index.ts            # Public exports
├── Card/
│   ├── Card.tsx
│   ├── CardHeader.tsx      # Sub-components
│   ├── CardBody.tsx
│   ├── CardFooter.tsx
│   ├── Card.test.tsx
│   └── index.ts
└── Form/
    ├── Form.tsx
    ├── FormField.tsx
    ├── FormError.tsx
    ├── useForm.ts          # Related hooks
    ├── validation.ts       # Related utilities
    └── index.ts
```

### API Layer Organization

```
core/api/
├── client.ts              # Axios/Fetch wrapper
├── interceptors.ts        # Request/response interceptors
├── types.ts              # API types
└── endpoints/
    ├── auth.ts           # Auth endpoints
    ├── products.ts       # Product endpoints
    ├── users.ts          # User endpoints
    └── index.ts

// Example client.ts
import axios from 'axios';

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Add interceptors
apiClient.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Example endpoint
// endpoints/products.ts
import { apiClient } from '../client';

export const productsApi = {
  getAll: () => apiClient.get('/products'),
  getById: (id: string) => apiClient.get(`/products/${id}`),
  create: (data: CreateProductDto) => apiClient.post('/products', data),
  update: (id: string, data: UpdateProductDto) =>
    apiClient.put(`/products/${id}`, data),
  delete: (id: string) => apiClient.delete(`/products/${id}`),
};
```

### Routing Structure

```
core/routing/
├── Router.tsx            # Main router component
├── routes.ts             # Route definitions
├── guards.ts             # Route guards
├── lazy-routes.ts        # Code-split routes
└── types.ts

// routes.ts
export const routes = {
  home: '/',
  auth: {
    login: '/login',
    register: '/register',
    resetPassword: '/reset-password',
  },
  dashboard: {
    root: '/dashboard',
    analytics: '/dashboard/analytics',
    settings: '/dashboard/settings',
  },
  products: {
    list: '/products',
    detail: '/products/:id',
    create: '/products/new',
    edit: '/products/:id/edit',
  },
} as const;

// Router.tsx
import { createBrowserRouter } from 'react-router-dom';
import { ProtectedRoute } from './guards';

export const router = createBrowserRouter([
  {
    path: routes.home,
    element: <AppLayout />,
    children: [
      { index: true, element: <HomePage /> },
      {
        path: routes.dashboard.root,
        element: <ProtectedRoute><DashboardLayout /></ProtectedRoute>,
        children: [
          { index: true, element: <DashboardHome /> },
          { path: routes.dashboard.analytics, element: <Analytics /> },
        ],
      },
    ],
  },
]);
```

### State Management Structure

```
store/
├── index.ts              # Store configuration
├── hooks.ts              # Typed hooks
├── types.ts              # Store types
├── middleware/           # Custom middleware
│   ├── logger.ts
│   └── analytics.ts
└── slices/               # Feature slices
    ├── authSlice.ts
    ├── cartSlice.ts
    └── productsSlice.ts

// index.ts
import { configureStore } from '@reduxjs/toolkit';
import authReducer from './slices/authSlice';
import cartReducer from './slices/cartSlice';

export const store = configureStore({
  reducer: {
    auth: authReducer,
    cart: cartReducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(logger),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// hooks.ts
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './index';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

### Type Definitions

```
types/
├── api/                  # API response types
│   ├── auth.ts
│   ├── products.ts
│   └── users.ts
├── models/               # Domain models
│   ├── User.ts
│   ├── Product.ts
│   └── Order.ts
├── components/           # Component prop types
│   └── common.ts
└── utils/                # Utility types
    └── helpers.ts

// Example: types/models/User.ts
export interface User {
  id: string;
  email: string;
  name: string;
  role: UserRole;
  createdAt: Date;
  updatedAt: Date;
}

export enum UserRole {
  Admin = 'admin',
  User = 'user',
  Guest = 'guest',
}

export type CreateUserDto = Omit<User, 'id' | 'createdAt' | 'updatedAt'>;
export type UpdateUserDto = Partial<CreateUserDto>;
```

## Barrel Exports

Use index files to control public API:

```typescript
// features/auth/index.ts
export { LoginForm } from './components/LoginForm';
export { RegisterForm } from './components/RegisterForm';
export { useAuth } from './hooks/useAuth';
export type { AuthState, LoginCredentials } from './types';

// Don't export internal implementation details
// authService remains private to the feature
```

## Path Aliases

Configure path aliases for clean imports:

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/shared/components/*"],
      "@features/*": ["src/features/*"],
      "@core/*": ["src/core/*"],
      "@hooks/*": ["src/shared/hooks/*"],
      "@utils/*": ["src/shared/utils/*"],
      "@types/*": ["src/types/*"]
    }
  }
}

// Usage
import { Button } from '@components/Button';
import { useAuth } from '@features/auth';
import { formatDate } from '@utils/date';
```

## Code Splitting Strategies

### 1. Route-Based Splitting

```tsx
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Products = lazy(() => import('./pages/Products'));
const Settings = lazy(() => import('./pages/Settings'));

const App = () => (
  <Suspense fallback={<PageLoader />}>
    <Routes>
      <Route path="/dashboard" element={<Dashboard />} />
      <Route path="/products" element={<Products />} />
      <Route path="/settings" element={<Settings />} />
    </Routes>
  </Suspense>
);
```

### 2. Feature-Based Splitting

```tsx
// Load entire feature module lazily
const AuthFeature = lazy(() => import('@features/auth'));
const DashboardFeature = lazy(() => import('@features/dashboard'));

// Features export their own routes
<Route path="/auth/*" element={<AuthFeature />} />
```

### 3. Component-Based Splitting

```tsx
// Heavy components loaded on demand
const RichTextEditor = lazy(() => import('@components/RichTextEditor'));
const DataVisualization = lazy(() => import('@components/Charts'));

const ArticleEditor = () => (
  <div>
    <Suspense fallback={<Skeleton />}>
      <RichTextEditor />
    </Suspense>
  </div>
);
```

## Environment Configuration

```
config/
├── env.ts                # Environment variables
├── constants.ts          # App constants
└── feature-flags.ts      # Feature toggles

// env.ts
export const env = {
  apiUrl: import.meta.env.VITE_API_URL,
  environment: import.meta.env.MODE,
  isDevelopment: import.meta.env.DEV,
  isProduction: import.meta.env.PROD,
  analytics: {
    enabled: import.meta.env.VITE_ANALYTICS_ENABLED === 'true',
    trackingId: import.meta.env.VITE_ANALYTICS_ID,
  },
} as const;

// constants.ts
export const APP_NAME = 'My App';
export const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
export const SUPPORTED_FORMATS = ['jpg', 'png', 'pdf'];

export const PAGINATION = {
  DEFAULT_PAGE_SIZE: 20,
  MAX_PAGE_SIZE: 100,
} as const;

// feature-flags.ts
export const featureFlags = {
  enableNewDashboard: true,
  enableBetaFeatures: env.isDevelopment,
  enableAnalytics: env.isProduction,
} as const;
```

## Asset Management

```
assets/
├── images/
│   ├── logo.svg
│   ├── icons/
│   └── illustrations/
├── fonts/
│   ├── inter/
│   └── roboto/
└── styles/
    ├── global.css
    ├── variables.css
    ├── reset.css
    └── themes/
        ├── light.css
        └── dark.css
```

## Testing Structure

```
src/
├── features/
│   └── auth/
│       ├── components/
│       │   ├── LoginForm.tsx
│       │   └── LoginForm.test.tsx
│       └── __tests__/
│           └── auth-integration.test.tsx
└── __tests__/
    ├── e2e/
    │   ├── auth.spec.ts
    │   └── products.spec.ts
    ├── integration/
    │   └── checkout-flow.test.tsx
    └── setup/
        ├── test-utils.tsx
        └── mocks.ts
```

## Monorepo Structure (Advanced)

For large organizations:

```
apps/
├── web/                  # Main web app
├── admin/                # Admin panel
└── mobile/               # React Native app

packages/
├── ui/                   # Shared UI components
├── utils/                # Shared utilities
├── types/                # Shared types
├── api-client/           # API client library
└── config/               # Shared configs

tools/
├── eslint-config/
└── typescript-config/
```

## Best Practices

### ✅ Do

- Use feature-based structure for scalability
- Implement path aliases
- Create barrel exports
- Split code by routes
- Organize by domain/feature
- Keep related files together
- Use consistent naming conventions
- Document structure decisions

### ❌ Don't

- Mix feature-based and layer-based randomly
- Create deeply nested folders
- Put everything in shared/common
- Export internal implementation details
- Ignore file naming conventions
- Create circular dependencies
- Over-organize too early

## Migration Strategy

### From Layer-Based to Feature-Based

1. **Identify features** - List business domains
2. **Create feature folders** - Set up new structure
3. **Move files incrementally** - One feature at a time
4. **Update imports** - Use path aliases
5. **Test thoroughly** - Ensure nothing breaks
6. **Remove old structure** - Clean up when done

```bash
# Example migration script
# Before: src/components/LoginForm.tsx
# After: src/features/auth/components/LoginForm.tsx

git mv src/components/LoginForm.tsx src/features/auth/components/
git mv src/hooks/useAuth.ts src/features/auth/hooks/
```

## Further Reading

- [Performance Architecture](./performance.md)
- [Micro-Frontends](./micro-frontends.md)
- [Design Systems](./design-systems.md)

---

*Next: [Performance Architecture →](./performance.md)*

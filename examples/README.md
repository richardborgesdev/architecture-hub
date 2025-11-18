# Real-World Examples

This section provides practical, runnable code examples demonstrating frontend architecture patterns and best practices.

## Available Examples

### Basic Patterns

#### 1. **Container/Presentational Pattern**
```
examples/container-presentational/
├── components/
│   ├── UserCard.tsx              # Presentational
│   └── UserCardContainer.tsx     # Container
└── README.md
```

**What you'll learn:**
- Separating logic from presentation
- Writing testable components
- Reusable UI components

---

#### 2. **Custom Hooks**
```
examples/custom-hooks/
├── hooks/
│   ├── useApi.ts
│   ├── useForm.ts
│   └── useLocalStorage.ts
└── examples/
    └── UserProfile.tsx
```

**What you'll learn:**
- Extracting reusable logic
- State management in hooks
- Side effects handling

---

#### 3. **Compound Components**
```
examples/compound-components/
├── components/
│   └── Tabs/
│       ├── Tabs.tsx
│       ├── TabList.tsx
│       ├── Tab.tsx
│       └── TabPanel.tsx
└── examples/
    └── TabsDemo.tsx
```

**What you'll learn:**
- Component composition
- Shared context
- Flexible APIs

---

### State Management

#### 4. **Redux Toolkit Setup**
```
examples/redux-toolkit/
├── store/
│   ├── store.ts
│   └── slices/
│       ├── authSlice.ts
│       └── cartSlice.ts
└── components/
    └── ShoppingCart.tsx
```

**What you'll learn:**
- Modern Redux setup
- Slice pattern
- Async actions with createAsyncThunk

---

#### 5. **React Query Implementation**
```
examples/react-query/
├── api/
│   └── client.ts
├── hooks/
│   ├── useUsers.ts
│   └── useProducts.ts
└── components/
    └── ProductList.tsx
```

**What you'll learn:**
- Server state management
- Caching strategies
- Optimistic updates

---

#### 6. **Zustand Store**
```
examples/zustand/
├── stores/
│   ├── userStore.ts
│   └── cartStore.ts
└── components/
    └── Cart.tsx
```

**What you'll learn:**
- Lightweight state management
- Store composition
- Middleware usage

---

### Application Structure

#### 7. **Feature-Based Structure**
```
examples/feature-based-structure/
├── features/
│   ├── auth/
│   ├── dashboard/
│   └── products/
├── shared/
└── core/
```

**What you'll learn:**
- Organizing large applications
- Feature isolation
- Code reusability

---

#### 8. **Micro-Frontends with Module Federation**
```
examples/micro-frontends/
├── container/
├── products-app/
└── checkout-app/
```

**What you'll learn:**
- Module Federation setup
- Independent deployment
- Shared dependencies

---

### Performance

#### 9. **Code Splitting & Lazy Loading**
```
examples/code-splitting/
├── routes/
│   └── lazyRoutes.tsx
└── components/
    └── LazyImage.tsx
```

**What you'll learn:**
- Route-based splitting
- Component lazy loading
- Suspense boundaries

---

#### 10. **Virtual Scrolling**
```
examples/virtual-scrolling/
├── components/
│   └── VirtualList.tsx
└── demo/
    └── LargeListDemo.tsx
```

**What you'll learn:**
- Rendering large lists
- React Window integration
- Performance optimization

---

#### 11. **Memoization Patterns**
```
examples/memoization/
├── components/
│   ├── MemoizedComponent.tsx
│   └── ExpensiveCalculation.tsx
└── hooks/
    └── useExpensiveValue.ts
```

**What you'll learn:**
- React.memo usage
- useMemo and useCallback
- When to optimize

---

### Design Systems

#### 12. **Component Library**
```
examples/component-library/
├── components/
│   ├── Button/
│   ├── Input/
│   └── Card/
├── tokens/
│   ├── colors.ts
│   └── typography.ts
└── stories/
    └── Button.stories.tsx
```

**What you'll learn:**
- Building reusable components
- Design tokens
- Storybook documentation

---

### Testing

#### 13. **Testing Patterns**
```
examples/testing/
├── components/
│   └── Button.test.tsx
├── hooks/
│   └── useForm.test.ts
└── integration/
    └── UserFlow.test.tsx
```

**What you'll learn:**
- Component testing
- Hook testing
- Integration testing

---

### Forms & Validation

#### 14. **Form Management**
```
examples/forms/
├── components/
│   └── RegistrationForm.tsx
├── hooks/
│   └── useForm.ts
└── validation/
    └── schemas.ts
```

**What you'll learn:**
- Form state management
- Validation patterns
- Error handling

---

### Routing

#### 15. **Advanced Routing**
```
examples/routing/
├── routes/
│   ├── ProtectedRoute.tsx
│   └── routes.ts
└── guards/
    └── authGuard.ts
```

**What you'll learn:**
- Route protection
- Dynamic routing
- Route guards

---

## Running Examples

### Prerequisites
```bash
node >= 18.0.0
npm >= 9.0.0
```

### Setup
```bash
# Clone the repository
git clone https://github.com/yourname/architecture-hub.git

# Navigate to an example
cd examples/container-presentational

# Install dependencies
npm install

# Run the example
npm run dev
```

## Example Structure

Each example includes:

```
example-name/
├── README.md          # Detailed explanation
├── src/              # Source code
├── package.json      # Dependencies
└── demo/             # Live demo code
```

## Learning Path

### For Beginners
1. Container/Presentational Pattern
2. Custom Hooks
3. Feature-Based Structure
4. Code Splitting

### For Intermediate
1. Redux Toolkit Setup
2. React Query Implementation
3. Compound Components
4. Testing Patterns

### For Advanced
1. Micro-Frontends
2. Virtual Scrolling
3. Performance Optimization
4. Design System Implementation

## Contributing Examples

Want to contribute an example? Follow these guidelines:

1. **Focus on one concept** - Keep examples focused
2. **Include README** - Explain what it demonstrates
3. **Add comments** - Explain why, not just what
4. **Keep it simple** - Avoid unnecessary complexity
5. **Make it runnable** - Ensure it works out of the box

### Example Template

```markdown
# Example Name

## What This Demonstrates
Brief description of the pattern/concept.

## Key Concepts
- Concept 1
- Concept 2
- Concept 3

## Running This Example
\`\`\`bash
npm install
npm run dev
\`\`\`

## Code Walkthrough
Explanation of the important parts.

## When to Use
Guidance on when this pattern is appropriate.

## Further Reading
Links to related concepts.
```

## Interactive Examples

Some examples include interactive demos:
- Stackblitz links
- CodeSandbox embeds
- Live preview links

## Questions?

If you have questions about any example:
1. Check the example's README
2. Review the related documentation
3. Open an issue on GitHub

---

*New examples are added regularly. Check back often!*

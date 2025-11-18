# Component Architecture

## Introduction

Component architecture is the foundation of modern frontend development. A well-designed component system enables teams to build consistent, maintainable, and scalable user interfaces.

## Component Design Principles

### 1. Single Responsibility Principle (SRP)

Each component should have one reason to change:

```tsx
// ❌ Bad: Multiple responsibilities
const UserProfile = () => {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);

  useEffect(() => {
    fetchUser();
    fetchPosts();
    trackAnalytics();
  }, []);

  return (
    <div>
      <UserHeader user={user} />
      <UserPosts posts={posts} />
      <UserComments userId={user?.id} />
    </div>
  );
};

// ✅ Good: Single responsibility
const UserProfile = ({ userId }) => {
  return (
    <div>
      <UserProfileHeader userId={userId} />
      <UserActivityFeed userId={userId} />
    </div>
  );
};

const UserProfileHeader = ({ userId }) => {
  const { data: user } = useUser(userId);
  return <UserHeader user={user} />;
};
```

### 2. Open/Closed Principle

Components should be open for extension, closed for modification:

```tsx
// ✅ Extensible through composition
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  icon?: ReactNode;
  children: ReactNode;
}

const Button = ({ variant = 'primary', size = 'md', icon, children }: ButtonProps) => {
  return (
    <button className={`btn btn-${variant} btn-${size}`}>
      {icon && <span className="btn-icon">{icon}</span>}
      {children}
    </button>
  );
};

// Extend through composition
const IconButton = ({ icon, ...props }) => (
  <Button icon={icon} {...props} />
);

const LoadingButton = ({ loading, children, ...props }) => (
  <Button {...props} disabled={loading}>
    {loading ? <Spinner /> : children}
  </Button>
);
```

### 3. Dependency Inversion

Depend on abstractions, not concrete implementations:

```tsx
// ✅ Using dependency injection
interface DataFetcher {
  fetch<T>(url: string): Promise<T>;
}

const UserList = ({ dataFetcher }: { dataFetcher: DataFetcher }) => {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    dataFetcher.fetch<User[]>('/api/users')
      .then(setUsers);
  }, []);

  return <List items={users} />;
};

// Can inject different implementations
<UserList dataFetcher={apiClient} />
<UserList dataFetcher={mockDataFetcher} />
```

## Component Taxonomy

### 1. Presentational Components

Pure UI components without business logic:

```tsx
interface CardProps {
  title: string;
  description: string;
  image?: string;
  actions?: ReactNode;
  className?: string;
}

export const Card = ({ title, description, image, actions, className }: CardProps) => {
  return (
    <div className={`card ${className}`}>
      {image && <img src={image} alt={title} className="card-image" />}
      <div className="card-content">
        <h3 className="card-title">{title}</h3>
        <p className="card-description">{description}</p>
      </div>
      {actions && <div className="card-actions">{actions}</div>}
    </div>
  );
};
```

**Characteristics:**
- Receives data via props
- Emits events via callbacks
- No state management
- Highly reusable
- Easy to test

### 2. Container Components

Handle data fetching and business logic:

```tsx
const UserProfileContainer = ({ userId }: { userId: string }) => {
  const { data: user, isLoading, error } = useUserQuery(userId);
  const updateUser = useUpdateUserMutation();

  const handleUpdate = async (updates: Partial<User>) => {
    await updateUser.mutateAsync({ userId, updates });
  };

  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;

  return (
    <UserProfile
      user={user}
      onUpdate={handleUpdate}
    />
  );
};
```

**Characteristics:**
- Manage state
- Handle side effects
- Connect to data sources
- Orchestrate logic
- Pass data to presentational components

### 3. Layout Components

Define page structure and responsive behavior:

```tsx
const PageLayout = ({ sidebar, header, children, footer }) => {
  return (
    <div className="page-layout">
      {header && <header className="page-header">{header}</header>}
      <div className="page-content">
        {sidebar && <aside className="page-sidebar">{sidebar}</aside>}
        <main className="page-main">{children}</main>
      </div>
      {footer && <footer className="page-footer">{footer}</footer>}
    </div>
  );
};

// Usage
<PageLayout
  header={<Header />}
  sidebar={<Navigation />}
  footer={<Footer />}
>
  <Dashboard />
</PageLayout>
```

### 4. Higher-Order Components (HOC)

Enhance components with additional functionality:

```tsx
// HOC for authentication
function withAuth<P extends object>(
  Component: ComponentType<P>
): ComponentType<P> {
  return (props: P) => {
    const { user, isAuthenticated } = useAuth();

    if (!isAuthenticated) {
      return <Navigate to="/login" />;
    }

    return <Component {...props} user={user} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard);
```

### 5. Render Props Pattern

Share logic through a render prop:

```tsx
interface MouseTrackerProps {
  children: (position: { x: number; y: number }) => ReactNode;
}

const MouseTracker = ({ children }: MouseTrackerProps) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (e: MouseEvent) => {
    setPosition({ x: e.clientX, y: e.clientY });
  };

  return (
    <div onMouseMove={handleMouseMove}>
      {children(position)}
    </div>
  );
};

// Usage
<MouseTracker>
  {({ x, y }) => (
    <div>Mouse is at ({x}, {y})</div>
  )}
</MouseTracker>
```

### 6. Custom Hooks (Modern Pattern)

Extract reusable logic:

```tsx
// Custom hook for data fetching
function useApi<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;

    fetch(url)
      .then(res => res.json())
      .then(data => {
        if (!cancelled) {
          setData(data);
          setLoading(false);
        }
      })
      .catch(err => {
        if (!cancelled) {
          setError(err);
          setLoading(false);
        }
      });

    return () => { cancelled = true; };
  }, [url]);

  return { data, loading, error };
}

// Usage in component
const UserList = () => {
  const { data: users, loading, error } = useApi<User[]>('/api/users');

  if (loading) return <Spinner />;
  if (error) return <Error message={error.message} />;

  return <List items={users} />;
};
```

## Component Composition Patterns

### 1. Compound Components

Components that work together to form a complete UI:

```tsx
// API similar to native HTML elements
const Tabs = ({ children, defaultValue }) => {
  const [activeTab, setActiveTab] = useState(defaultValue);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      {children}
    </TabsContext.Provider>
  );
};

Tabs.List = ({ children }) => (
  <div className="tabs-list">{children}</div>
);

Tabs.Tab = ({ value, children }) => {
  const { activeTab, setActiveTab } = useTabsContext();
  return (
    <button
      className={activeTab === value ? 'active' : ''}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
};

Tabs.Panel = ({ value, children }) => {
  const { activeTab } = useTabsContext();
  return activeTab === value ? <div>{children}</div> : null;
};

// Usage
<Tabs defaultValue="profile">
  <Tabs.List>
    <Tabs.Tab value="profile">Profile</Tabs.Tab>
    <Tabs.Tab value="settings">Settings</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel value="profile"><ProfileContent /></Tabs.Panel>
  <Tabs.Panel value="settings"><SettingsContent /></Tabs.Panel>
</Tabs>
```

### 2. Controlled vs Uncontrolled Components

```tsx
// Uncontrolled - component manages own state
const UncontrolledInput = () => {
  const inputRef = useRef<HTMLInputElement>(null);

  const handleSubmit = () => {
    console.log(inputRef.current?.value);
  };

  return <input ref={inputRef} />;
};

// Controlled - parent manages state
const ControlledInput = ({ value, onChange }) => {
  return <input value={value} onChange={e => onChange(e.target.value)} />;
};

// Usage
const Form = () => {
  const [name, setName] = useState('');

  return <ControlledInput value={name} onChange={setName} />;
};
```

### 3. Slots Pattern

Flexible content areas:

```tsx
interface DialogProps {
  header?: ReactNode;
  content: ReactNode;
  footer?: ReactNode;
  isOpen: boolean;
  onClose: () => void;
}

const Dialog = ({ header, content, footer, isOpen, onClose }: DialogProps) => {
  if (!isOpen) return null;

  return (
    <div className="dialog-overlay" onClick={onClose}>
      <div className="dialog" onClick={e => e.stopPropagation()}>
        {header && <div className="dialog-header">{header}</div>}
        <div className="dialog-content">{content}</div>
        {footer && <div className="dialog-footer">{footer}</div>}
      </div>
    </div>
  );
};

// Usage
<Dialog
  isOpen={isOpen}
  onClose={handleClose}
  header={<h2>Confirm Action</h2>}
  content={<p>Are you sure?</p>}
  footer={
    <>
      <Button onClick={handleConfirm}>Confirm</Button>
      <Button onClick={handleClose}>Cancel</Button>
    </>
  }
/>
```

## Component Communication

### 1. Props Down

Parent → Child communication:

```tsx
<UserCard user={user} onEdit={handleEdit} />
```

### 2. Callbacks Up

Child → Parent communication:

```tsx
const SearchInput = ({ onSearch }) => {
  const [query, setQuery] = useState('');

  const handleSubmit = () => {
    onSearch(query);
  };

  return (
    <input value={query} onChange={e => setQuery(e.target.value)} />
  );
};
```

### 3. Context for Deep Trees

Avoid prop drilling:

```tsx
const ThemeContext = createContext<Theme>(defaultTheme);

const ThemeProvider = ({ children, theme }) => (
  <ThemeContext.Provider value={theme}>
    {children}
  </ThemeContext.Provider>
);

const useTheme = () => useContext(ThemeContext);

// Usage anywhere in the tree
const Button = () => {
  const theme = useTheme();
  return <button style={{ color: theme.primaryColor }}>Click</button>;
};
```

### 4. Event Emitters

For loosely coupled components:

```tsx
// Event bus
class EventBus {
  private listeners = new Map();

  on(event: string, callback: Function) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event).push(callback);
  }

  emit(event: string, data: any) {
    this.listeners.get(event)?.forEach(cb => cb(data));
  }
}

export const eventBus = new EventBus();

// Component A
eventBus.emit('user:updated', user);

// Component B
useEffect(() => {
  eventBus.on('user:updated', handleUserUpdate);
}, []);
```

## Performance Optimization

### 1. Memoization

```tsx
// Prevent unnecessary re-renders
const ExpensiveComponent = memo(({ data }) => {
  return <div>{/* expensive rendering */}</div>;
});

// Memoize calculations
const ProcessedData = ({ items }) => {
  const sortedItems = useMemo(
    () => items.sort((a, b) => a.value - b.value),
    [items]
  );

  return <List items={sortedItems} />;
};

// Memoize callbacks
const ItemList = ({ onItemClick }) => {
  const handleClick = useCallback(
    (id) => onItemClick(id),
    [onItemClick]
  );

  return items.map(item => (
    <Item key={item.id} onClick={() => handleClick(item.id)} />
  ));
};
```

### 2. Code Splitting

```tsx
// Lazy load components
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => import('./Settings'));

const App = () => (
  <Suspense fallback={<Loading />}>
    <Routes>
      <Route path="/dashboard" element={<Dashboard />} />
      <Route path="/settings" element={<Settings />} />
    </Routes>
  </Suspense>
);
```

### 3. Virtual Scrolling

For large lists:

```tsx
import { FixedSizeList } from 'react-window';

const VirtualizedList = ({ items }) => (
  <FixedSizeList
    height={600}
    itemCount={items.length}
    itemSize={50}
    width="100%"
  >
    {({ index, style }) => (
      <div style={style}>{items[index].name}</div>
    )}
  </FixedSizeList>
);
```

## Testing Strategy

### Unit Tests

```tsx
import { render, screen, fireEvent } from '@testing-library/react';

describe('Button', () => {
  it('renders with children', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click</Button>);
    fireEvent.click(screen.getByText('Click'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

### Integration Tests

```tsx
describe('UserProfile', () => {
  it('displays user data after loading', async () => {
    const mockUser = { id: '1', name: 'John Doe' };
    mockApi.getUser.mockResolvedValue(mockUser);

    render(<UserProfile userId="1" />);

    expect(screen.getByText('Loading...')).toBeInTheDocument();

    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });
  });
});
```

## Best Practices

### ✅ Do

- Keep components small and focused
- Use TypeScript for type safety
- Document complex components
- Write tests for business logic
- Use composition over inheritance
- Follow consistent naming conventions
- Extract reusable logic into hooks
- Optimize performance with profiling first

### ❌ Don't

- Create god components
- Over-abstract prematurely
- Forget error boundaries
- Ignore accessibility
- Skip prop validation
- Use inline functions in JSX (when it matters)
- Mutate props
- Create deeply nested component trees

## Further Reading

- [State Management Strategies](./state-management.md)
- [Performance Architecture](./performance.md)
- [Design Systems](./design-systems.md)

---

*Next: [State Management →](./state-management.md)*

# Frontend-Specific Design Patterns

## Introduction

Frontend patterns are proven solutions to common problems in user interface development. This guide focuses on patterns specifically applicable to modern frontend frameworks like React, Vue, and Angular.

## Component Patterns

### 1. Container/Presentational Pattern

Separate logic from presentation.

```tsx
// Presentational Component (Dumb)
interface UserCardProps {
  user: User;
  onEdit: (id: string) => void;
  onDelete: (id: string) => void;
}

export const UserCard = ({ user, onEdit, onDelete }: UserCardProps) => {
  return (
    <div className="user-card">
      <img src={user.avatar} alt={user.name} />
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={() => onEdit(user.id)}>Edit</button>
      <button onClick={() => onDelete(user.id)}>Delete</button>
    </div>
  );
};

// Container Component (Smart)
export const UserCardContainer = ({ userId }: { userId: string }) => {
  const { data: user, isLoading } = useUser(userId);
  const deleteUser = useDeleteUser();
  const navigate = useNavigate();

  const handleEdit = (id: string) => {
    navigate(`/users/${id}/edit`);
  };

  const handleDelete = async (id: string) => {
    await deleteUser.mutateAsync(id);
  };

  if (isLoading) return <Skeleton />;
  if (!user) return <NotFound />;

  return (
    <UserCard
      user={user}
      onEdit={handleEdit}
      onDelete={handleDelete}
    />
  );
};
```

**Benefits:**
- Clear separation of concerns
- Highly testable presentational components
- Reusable UI components
- Easy to understand data flow

**When to use:**
- Complex components with business logic
- When you need reusable UI components
- Testing is a priority

### 2. Compound Components

Components that work together to form a complete UI.

```tsx
// Create context for internal communication
const TabsContext = createContext<{
  activeTab: string;
  setActiveTab: (tab: string) => void;
} | null>(null);

// Main component
export const Tabs = ({ children, defaultValue }: TabsProps) => {
  const [activeTab, setActiveTab] = useState(defaultValue);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
};

// Sub-components
Tabs.List = ({ children }: { children: ReactNode }) => (
  <div className="tabs-list" role="tablist">
    {children}
  </div>
);

Tabs.Tab = ({ value, children }: TabProps) => {
  const context = useContext(TabsContext);
  if (!context) throw new Error('Tab must be used within Tabs');

  const { activeTab, setActiveTab } = context;
  const isActive = activeTab === value;

  return (
    <button
      role="tab"
      aria-selected={isActive}
      className={`tab ${isActive ? 'active' : ''}`}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
};

Tabs.Panel = ({ value, children }: TabPanelProps) => {
  const context = useContext(TabsContext);
  if (!context) throw new Error('TabPanel must be used within Tabs');

  const { activeTab } = context;

  if (activeTab !== value) return null;

  return (
    <div role="tabpanel" className="tab-panel">
      {children}
    </div>
  );
};

// Usage
<Tabs defaultValue="profile">
  <Tabs.List>
    <Tabs.Tab value="profile">Profile</Tabs.Tab>
    <Tabs.Tab value="settings">Settings</Tabs.Tab>
    <Tabs.Tab value="notifications">Notifications</Tabs.Tab>
  </Tabs.List>

  <Tabs.Panel value="profile">
    <ProfileContent />
  </Tabs.Panel>
  <Tabs.Panel value="settings">
    <SettingsContent />
  </Tabs.Panel>
  <Tabs.Panel value="notifications">
    <NotificationsContent />
  </Tabs.Panel>
</Tabs>
```

**Benefits:**
- Flexible and declarative API
- Shared state between components
- Intuitive component relationships
- Similar to native HTML elements

**When to use:**
- Complex UI components (tabs, accordions, dropdowns)
- Components with shared state
- Building component libraries

### 3. Render Props Pattern

Share code between components using a prop whose value is a function.

```tsx
interface MouseTrackerProps {
  children: (position: { x: number; y: number }) => ReactNode;
}

export const MouseTracker = ({ children }: MouseTrackerProps) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (e: React.MouseEvent) => {
    setPosition({ x: e.clientX, y: e.clientY });
  };

  return (
    <div
      onMouseMove={handleMouseMove}
      style={{ height: '100vh' }}
    >
      {children(position)}
    </div>
  );
};

// Usage
<MouseTracker>
  {({ x, y }) => (
    <div>
      <h2>Mouse position:</h2>
      <p>X: {x}, Y: {y}</p>
    </div>
  )}
</MouseTracker>

// Another usage
<MouseTracker>
  {({ x, y }) => (
    <img
      src="cursor.png"
      style={{
        position: 'absolute',
        left: x,
        top: y
      }}
    />
  )}
</MouseTracker>
```

**Benefits:**
- Highly reusable logic
- Flexible rendering
- Explicit data flow

**When to use:**
- Sharing stateful logic
- Need different renderings of same data
- Alternative to HOCs

**Modern alternative:**
```tsx
// Custom hook (preferred in React)
const useMousePosition = () => {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };

    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);

  return position;
};

// Usage
const Component = () => {
  const { x, y } = useMousePosition();
  return <div>X: {x}, Y: {y}</div>;
};
```

### 4. Higher-Order Component (HOC)

Function that takes a component and returns a new component.

```tsx
// HOC for authentication
function withAuth<P extends object>(
  Component: ComponentType<P & { user: User }>
) {
  return function AuthenticatedComponent(props: P) {
    const { user, isLoading } = useAuth();
    const navigate = useNavigate();

    useEffect(() => {
      if (!isLoading && !user) {
        navigate('/login');
      }
    }, [user, isLoading, navigate]);

    if (isLoading) return <LoadingSpinner />;
    if (!user) return null;

    return <Component {...props} user={user} />;
  };
}

// Usage
const Dashboard = ({ user }: { user: User }) => {
  return <div>Welcome, {user.name}!</div>;
};

export default withAuth(Dashboard);

// HOC for logging
function withLogging<P extends object>(
  Component: ComponentType<P>,
  componentName: string
) {
  return function LoggedComponent(props: P) {
    useEffect(() => {
      console.log(`${componentName} mounted`);
      return () => console.log(`${componentName} unmounted`);
    }, []);

    return <Component {...props} />;
  };
}

// Compose multiple HOCs
const EnhancedDashboard = withLogging(withAuth(Dashboard), 'Dashboard');
```

**Benefits:**
- Reusable cross-cutting concerns
- Props enhancement
- Composition pattern

**Drawbacks:**
- Props collision
- Wrapper hell
- Ref forwarding issues

**When to use:**
- Cross-cutting concerns (auth, logging, error handling)
- Enhance existing components
- Props manipulation

**Modern alternative:** Custom hooks are usually preferred.

### 5. Custom Hooks Pattern

Extract and reuse stateful logic.

```tsx
// Reusable data fetching hook
function useApi<T>(url: string, options?: RequestInit) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;

    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url, options);
        const json = await response.json();

        if (!cancelled) {
          setData(json);
          setError(null);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err as Error);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    };

    fetchData();

    return () => {
      cancelled = true;
    };
  }, [url]);

  return { data, loading, error };
}

// Usage
const UserProfile = ({ userId }: { userId: string }) => {
  const { data: user, loading, error } = useApi<User>(`/api/users/${userId}`);

  if (loading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  if (!user) return <NotFound />;

  return <Profile user={user} />;
};

// Reusable form hook
function useForm<T extends Record<string, any>>(initialValues: T) {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});

  const handleChange = (name: keyof T) => (
    e: React.ChangeEvent<HTMLInputElement>
  ) => {
    setValues(prev => ({
      ...prev,
      [name]: e.target.value,
    }));
  };

  const handleBlur = (name: keyof T) => () => {
    setTouched(prev => ({
      ...prev,
      [name]: true,
    }));
  };

  const reset = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  };

  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    setFieldValue: (name: keyof T, value: any) => {
      setValues(prev => ({ ...prev, [name]: value }));
    },
    setFieldError: (name: keyof T, error: string) => {
      setErrors(prev => ({ ...prev, [name]: error }));
    },
    reset,
  };
}

// Usage
const LoginForm = () => {
  const form = useForm({
    email: '',
    password: '',
  });

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log(form.values);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={form.values.email}
        onChange={form.handleChange('email')}
        onBlur={form.handleBlur('email')}
      />
      <input
        type="password"
        value={form.values.password}
        onChange={form.handleChange('password')}
        onBlur={form.handleBlur('password')}
      />
      <button type="submit">Login</button>
    </form>
  );
};
```

**Benefits:**
- Excellent code reuse
- Separation of concerns
- Composable
- Testing is straightforward

**When to use:**
- Extracting component logic
- Sharing stateful logic
- Side effects management

## State Management Patterns

### 6. Flux/Redux Pattern

Unidirectional data flow.

```tsx
// Actions
const INCREMENT = 'counter/increment';
const DECREMENT = 'counter/decrement';

// Action creators
const increment = () => ({ type: INCREMENT as const });
const decrement = () => ({ type: DECREMENT as const });

type Action = ReturnType<typeof increment | typeof decrement>;

// Reducer
interface State {
  count: number;
}

const initialState: State = { count: 0 };

const counterReducer = (state = initialState, action: Action): State => {
  switch (action.type) {
    case INCREMENT:
      return { count: state.count + 1 };
    case DECREMENT:
      return { count: state.count - 1 };
    default:
      return state;
  }
};

// Component
const Counter = () => {
  const count = useSelector((state: RootState) => state.counter.count);
  const dispatch = useDispatch();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
    </div>
  );
};
```

### 7. Observer Pattern

Subscribe to state changes.

```tsx
// Simple event emitter
class EventEmitter<T = any> {
  private listeners: Map<string, Set<(data: T) => void>> = new Map();

  on(event: string, callback: (data: T) => void) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(callback);
  }

  off(event: string, callback: (data: T) => void) {
    this.listeners.get(event)?.delete(callback);
  }

  emit(event: string, data: T) {
    this.listeners.get(event)?.forEach(callback => callback(data));
  }
}

// Usage
const userEvents = new EventEmitter<User>();

// Component A - emits
const UserForm = () => {
  const handleSave = (user: User) => {
    userEvents.emit('user:updated', user);
  };
};

// Component B - listens
const UserList = () => {
  useEffect(() => {
    const handleUserUpdate = (user: User) => {
      console.log('User updated:', user);
    };

    userEvents.on('user:updated', handleUserUpdate);

    return () => {
      userEvents.off('user:updated', handleUserUpdate);
    };
  }, []);
};
```

### 8. Provider Pattern

Share data without prop drilling.

```tsx
// Create context with default value
const ThemeContext = createContext<{
  theme: 'light' | 'dark';
  toggleTheme: () => void;
} | null>(null);

// Provider component
export const ThemeProvider = ({ children }: { children: ReactNode }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// Custom hook for consuming
export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};

// Usage
const App = () => (
  <ThemeProvider>
    <Layout />
  </ThemeProvider>
);

const Button = () => {
  const { theme, toggleTheme } = useTheme();
  return (
    <button
      className={`btn-${theme}`}
      onClick={toggleTheme}
    >
      Toggle Theme
    </button>
  );
};
```

## Performance Patterns

### 9. Memoization Pattern

Cache expensive computations.

```tsx
// Memoize component
const ExpensiveList = memo(({ items }: { items: Item[] }) => {
  console.log('Rendering list...');
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
});

// Memoize value
const Dashboard = ({ data }: { data: Data[] }) => {
  const processedData = useMemo(() => {
    console.log('Processing data...');
    return data
      .filter(item => item.active)
      .sort((a, b) => b.value - a.value)
      .slice(0, 10);
  }, [data]);

  return <Chart data={processedData} />;
};

// Memoize callback
const List = ({ items, onItemClick }: ListProps) => {
  const handleClick = useCallback(
    (id: string) => {
      onItemClick(id);
    },
    [onItemClick]
  );

  return (
    <>
      {items.map(item => (
        <Item
          key={item.id}
          item={item}
          onClick={handleClick}
        />
      ))}
    </>
  );
};
```

### 10. Lazy Loading Pattern

Load resources on demand.

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

// Lazy load images
const LazyImage = ({ src, alt }: { src: string; alt: string }) => {
  const [imageSrc, setImageSrc] = useState<string>('');
  const imgRef = useRef<HTMLImageElement>(null);

  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        setImageSrc(src);
        observer.disconnect();
      }
    });

    if (imgRef.current) {
      observer.observe(imgRef.current);
    }

    return () => observer.disconnect();
  }, [src]);

  return <img ref={imgRef} src={imageSrc} alt={alt} loading="lazy" />;
};
```

## Further Reading

- [Component Architecture](../frontend/component-architecture.md)
- [State Management](../frontend/state-management.md)
- [Performance Architecture](../frontend/performance.md)

---

*See also: [Creational Patterns](./creational.md) | [Structural Patterns](./structural.md) | [Behavioral Patterns](./behavioral.md)*

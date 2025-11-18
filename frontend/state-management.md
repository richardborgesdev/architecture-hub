# State Management

## Introduction

State management is one of the most critical architectural decisions in frontend applications. The right state management strategy can make your application maintainable and scalable, while the wrong choice can lead to complexity and bugs.

## Types of State

### 1. Local Component State

State that only affects a single component:

```tsx
const Counter = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};
```

**When to use:**
- UI state (toggles, modals, form inputs)
- Component-specific data
- Temporary calculations

### 2. Shared State (Lifted State)

State shared between sibling components:

```tsx
const Parent = () => {
  const [selectedId, setSelectedId] = useState(null);

  return (
    <>
      <Sidebar onSelect={setSelectedId} selectedId={selectedId} />
      <Content selectedId={selectedId} />
    </>
  );
};
```

**When to use:**
- Data needed by multiple components
- Communication between siblings
- Simple parent-child data flow

### 3. Global State

Application-wide state accessible anywhere:

```tsx
// Using Context
const AppStateContext = createContext();

const AppProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');

  return (
    <AppStateContext.Provider value={{ user, setUser, theme, setTheme }}>
      {children}
    </AppStateContext.Provider>
  );
};
```

**When to use:**
- User authentication
- Theme/i18n settings
- App-wide configuration
- Shopping cart, notifications

### 4. Server State

Data synchronized with backend:

```tsx
// Using React Query
const UserProfile = ({ userId }) => {
  const { data, isLoading, error } = useQuery(
    ['user', userId],
    () => fetchUser(userId),
    {
      staleTime: 5000,
      cacheTime: 10000,
    }
  );

  if (isLoading) return <Spinner />;
  if (error) return <Error />;

  return <Profile user={data} />;
};
```

**When to use:**
- API data
- Real-time updates
- Cached backend data
- Optimistic updates

### 5. URL State

State stored in URL parameters:

```tsx
const SearchPage = () => {
  const [searchParams, setSearchParams] = useSearchParams();
  const query = searchParams.get('q') || '';
  const page = parseInt(searchParams.get('page') || '1');

  const handleSearch = (newQuery) => {
    setSearchParams({ q: newQuery, page: '1' });
  };

  return <SearchResults query={query} page={page} onSearch={handleSearch} />;
};
```

**When to use:**
- Search filters
- Pagination
- Tab selection
- Shareable application state

## State Management Solutions

### 1. React Context + Hooks

**Pros:**
- Built-in, no dependencies
- Simple for small to medium apps
- Good for theme, auth, i18n

**Cons:**
- Can cause unnecessary re-renders
- No built-in devtools
- Can become verbose for complex state

```tsx
// Create context with reducer
interface State {
  count: number;
  user: User | null;
}

type Action =
  | { type: 'INCREMENT' }
  | { type: 'SET_USER'; payload: User };

const StateContext = createContext<{
  state: State;
  dispatch: Dispatch<Action>;
} | null>(null);

const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'SET_USER':
      return { ...state, user: action.payload };
    default:
      return state;
  }
};

const StateProvider = ({ children }) => {
  const [state, dispatch] = useReducer(reducer, {
    count: 0,
    user: null,
  });

  return (
    <StateContext.Provider value={{ state, dispatch }}>
      {children}
    </StateContext.Provider>
  );
};

const useAppState = () => {
  const context = useContext(StateContext);
  if (!context) throw new Error('useAppState must be used within StateProvider');
  return context;
};
```

### 2. Redux Toolkit

**Pros:**
- Predictable state updates
- Excellent devtools
- Large ecosystem
- Great for complex applications

**Cons:**
- Boilerplate (reduced with RTK)
- Learning curve
- Overkill for simple apps

```tsx
// Store setup
import { configureStore, createSlice } from '@reduxjs/toolkit';

const userSlice = createSlice({
  name: 'user',
  initialState: { data: null, loading: false, error: null },
  reducers: {
    setUser: (state, action) => {
      state.data = action.payload;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.data = action.payload;
        state.loading = false;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.error = action.error.message;
        state.loading = false;
      });
  },
});

const store = configureStore({
  reducer: {
    user: userSlice.reducer,
  },
});

// Component usage
const UserProfile = () => {
  const user = useSelector((state: RootState) => state.user.data);
  const dispatch = useDispatch();

  useEffect(() => {
    dispatch(fetchUser('123'));
  }, []);

  return <div>{user?.name}</div>;
};
```

### 3. Zustand

**Pros:**
- Minimal boilerplate
- Great TypeScript support
- No providers needed
- Small bundle size

**Cons:**
- Less mature ecosystem
- Fewer devtools features

```tsx
import create from 'zustand';

interface UserStore {
  user: User | null;
  setUser: (user: User) => void;
  fetchUser: (id: string) => Promise<void>;
}

const useUserStore = create<UserStore>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
  fetchUser: async (id) => {
    const user = await api.getUser(id);
    set({ user });
  },
}));

// Component usage
const UserProfile = () => {
  const { user, fetchUser } = useUserStore();

  useEffect(() => {
    fetchUser('123');
  }, []);

  return <div>{user?.name}</div>;
};
```

### 4. MobX

**Pros:**
- Reactive programming
- Less boilerplate
- Automatic tracking
- Good for OOP background

**Cons:**
- Magic can be confusing
- Steeper learning curve
- Less predictable

```tsx
import { makeAutoObservable } from 'mobx';
import { observer } from 'mobx-react-lite';

class UserStore {
  user: User | null = null;
  loading = false;

  constructor() {
    makeAutoObservable(this);
  }

  async fetchUser(id: string) {
    this.loading = true;
    this.user = await api.getUser(id);
    this.loading = false;
  }
}

const userStore = new UserStore();

// Component usage
const UserProfile = observer(() => {
  useEffect(() => {
    userStore.fetchUser('123');
  }, []);

  if (userStore.loading) return <Spinner />;
  return <div>{userStore.user?.name}</div>;
});
```

### 5. Jotai (Atomic State)

**Pros:**
- Minimal API
- Bottom-up approach
- Great for derived state
- TypeScript friendly

**Cons:**
- Different mental model
- Newer library

```tsx
import { atom, useAtom } from 'jotai';

// Atoms
const userAtom = atom<User | null>(null);
const userNameAtom = atom((get) => get(userAtom)?.name ?? 'Guest');

// Component usage
const UserGreeting = () => {
  const [userName] = useAtom(userNameAtom);
  return <h1>Hello, {userName}!</h1>;
};

const UserProfile = () => {
  const [user, setUser] = useAtom(userAtom);

  useEffect(() => {
    api.getUser('123').then(setUser);
  }, []);

  return <div>{user?.email}</div>;
};
```

### 6. React Query / TanStack Query

**Pros:**
- Perfect for server state
- Built-in caching
- Automatic refetching
- Optimistic updates

**Cons:**
- Only for async/server state
- Needs separate client state solution

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Query
const useUser = (userId: string) => {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => api.getUser(userId),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};

// Mutation
const useUpdateUser = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: { userId: string; updates: Partial<User> }) =>
      api.updateUser(data.userId, data.updates),
    onSuccess: (data, variables) => {
      // Invalidate and refetch
      queryClient.invalidateQueries(['user', variables.userId]);
    },
  });
};

// Component usage
const UserProfile = ({ userId }) => {
  const { data: user, isLoading, error } = useUser(userId);
  const updateUser = useUpdateUser();

  const handleUpdate = (updates: Partial<User>) => {
    updateUser.mutate({ userId, updates });
  };

  if (isLoading) return <Spinner />;
  if (error) return <Error message={error.message} />;

  return <Profile user={user} onUpdate={handleUpdate} />;
};
```

## State Management Patterns

### 1. Container/Presenter Pattern

Separate state management from presentation:

```tsx
// Container (handles state)
const UserDashboardContainer = () => {
  const { data: user, isLoading } = useUser();
  const { data: posts } = usePosts(user?.id);

  if (isLoading) return <Spinner />;

  return <UserDashboard user={user} posts={posts} />;
};

// Presenter (pure UI)
const UserDashboard = ({ user, posts }) => {
  return (
    <div>
      <UserHeader user={user} />
      <PostList posts={posts} />
    </div>
  );
};
```

### 2. Flux Pattern (Unidirectional Data Flow)

```
Action → Dispatcher → Store → View → Action
```

```tsx
// Action creators
const actions = {
  addTodo: (text: string) => ({ type: 'ADD_TODO', payload: text }),
  toggleTodo: (id: string) => ({ type: 'TOGGLE_TODO', payload: id }),
};

// Reducer
const todoReducer = (state: Todo[], action: Action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, { id: uuid(), text: action.payload, done: false }];
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.payload ? { ...todo, done: !todo.done } : todo
      );
    default:
      return state;
  }
};

// Store
const [todos, dispatch] = useReducer(todoReducer, []);

// View
<button onClick={() => dispatch(actions.addTodo('New task'))}>
  Add Todo
</button>
```

### 3. Domain-Driven Design

Organize state by domain:

```tsx
// store/
├── auth/
│   ├── authSlice.ts
│   ├── authSelectors.ts
│   └── authHooks.ts
├── products/
│   ├── productsSlice.ts
│   ├── productsSelectors.ts
│   └── productsHooks.ts
└── cart/
    ├── cartSlice.ts
    ├── cartSelectors.ts
    └── cartHooks.ts
```

### 4. Optimistic Updates

Update UI immediately, rollback on error:

```tsx
const useUpdateTodo = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: updateTodo,
    onMutate: async (newTodo) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries(['todos']);

      // Snapshot previous value
      const previousTodos = queryClient.getQueryData(['todos']);

      // Optimistically update
      queryClient.setQueryData(['todos'], (old: Todo[]) =>
        old.map(todo => todo.id === newTodo.id ? newTodo : todo)
      );

      return { previousTodos };
    },
    onError: (err, newTodo, context) => {
      // Rollback on error
      queryClient.setQueryData(['todos'], context.previousTodos);
    },
    onSettled: () => {
      // Refetch after error or success
      queryClient.invalidateQueries(['todos']);
    },
  });
};
```

### 5. Normalized State

For complex relational data:

```tsx
// Instead of nested data
const badState = {
  posts: [
    {
      id: '1',
      author: { id: 'a1', name: 'John' },
      comments: [
        { id: 'c1', author: { id: 'a2', name: 'Jane' } }
      ]
    }
  ]
};

// Normalize it
const goodState = {
  posts: {
    byId: {
      '1': { id: '1', authorId: 'a1', commentIds: ['c1'] }
    },
    allIds: ['1']
  },
  users: {
    byId: {
      'a1': { id: 'a1', name: 'John' },
      'a2': { id: 'a2', name: 'Jane' }
    },
    allIds: ['a1', 'a2']
  },
  comments: {
    byId: {
      'c1': { id: 'c1', authorId: 'a2' }
    },
    allIds: ['c1']
  }
};

// Benefits: Easy updates, no duplication, better performance
```

## Decision Framework

### Choose Local State When:
- State is only used in one component
- Simple UI interactions (toggles, form inputs)
- No need to share with other components

### Choose Context When:
- Need to share state across component tree
- Avoid prop drilling
- Theme, auth, i18n
- Small to medium apps

### Choose Redux/Zustand When:
- Complex state logic
- Many components need same state
- Time-travel debugging needed
- Large team, need predictability

### Choose React Query When:
- Managing server state
- Need caching and synchronization
- Optimistic updates
- Automatic background refetching

### Choose MobX When:
- Team has OOP background
- Need reactive programming
- Complex derived state
- Want less boilerplate

## Performance Considerations

### 1. Avoid Unnecessary Re-renders

```tsx
// ❌ Bad: Creates new object every render
const Component = () => {
  const value = { user: currentUser };
  return <Context.Provider value={value}>...</Context.Provider>;
};

// ✅ Good: Memoize the value
const Component = () => {
  const value = useMemo(() => ({ user: currentUser }), [currentUser]);
  return <Context.Provider value={value}>...</Context.Provider>;
};
```

### 2. Selector Optimization

```tsx
// ❌ Bad: Returns new array every time
const useFilteredTodos = () => {
  const todos = useSelector(state => state.todos.filter(t => !t.done));
  return todos;
};

// ✅ Good: Use memoized selector
import { createSelector } from 'reselect';

const selectTodos = state => state.todos;
const selectActiveTodos = createSelector(
  [selectTodos],
  todos => todos.filter(t => !t.done)
);

const useActiveTodos = () => useSelector(selectActiveTodos);
```

### 3. Code Splitting State

```tsx
// Lazy load store modules
const dashboardSlice = lazy(() => import('./features/dashboard/dashboardSlice'));

// Only load when needed
if (isDashboardRoute) {
  store.injectReducer('dashboard', dashboardSlice);
}
```

## Best Practices

### ✅ Do

- Keep state as local as possible
- Separate client and server state
- Use TypeScript for type safety
- Normalize complex relational data
- Implement error handling
- Cache API responses
- Use selectors for derived state
- Document state shape and flow

### ❌ Don't

- Put everything in global state
- Store derived data in state
- Mutate state directly
- Use context for high-frequency updates
- Ignore performance implications
- Mix state management solutions unnecessarily
- Store what can be calculated

## Further Reading

- [Application Structure](./app-structure.md)
- [Performance Architecture](./performance.md)
- [Frontend Patterns](../patterns/frontend-patterns.md)

---

*Next: [Application Structure →](./app-structure.md)*

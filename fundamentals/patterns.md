# Architecture Patterns

## Introduction

Architecture patterns are proven, reusable solutions to common architectural problems. Understanding these patterns helps you make informed decisions about how to structure your applications.

## Layered Architecture (N-Tier)

### Overview

Organizes code into horizontal layers, each with a specific responsibility.

```
┌─────────────────────────┐
│   Presentation Layer    │  (UI, Controllers)
├─────────────────────────┤
│   Business Logic Layer  │  (Domain, Services)
├─────────────────────────┤
│   Data Access Layer     │  (Repositories, ORM)
├─────────────────────────┤
│   Database Layer        │  (SQL, NoSQL)
└─────────────────────────┘
```

### Example

```typescript
// Presentation Layer
class UserController {
  constructor(private userService: UserService) {}

  async getUser(req: Request, res: Response) {
    const user = await this.userService.getUserById(req.params.id);
    res.json(user);
  }
}

// Business Logic Layer
class UserService {
  constructor(private userRepository: UserRepository) {}

  async getUserById(id: string): Promise<User> {
    const user = await this.userRepository.findById(id);
    if (!user) throw new Error('User not found');
    return user;
  }

  async updateUser(id: string, data: Partial<User>): Promise<User> {
    // Business logic validation
    if (data.email && !this.isValidEmail(data.email)) {
      throw new Error('Invalid email');
    }
    return this.userRepository.update(id, data);
  }

  private isValidEmail(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
}

// Data Access Layer
class UserRepository {
  async findById(id: string): Promise<User | null> {
    return db.users.findOne({ id });
  }

  async update(id: string, data: Partial<User>): Promise<User> {
    return db.users.updateOne({ id }, data);
  }
}
```

**Pros:**
- Clear separation of concerns
- Easy to understand
- Good for traditional web applications
- Team members can work on different layers

**Cons:**
- Can become monolithic
- Changes often require updates across layers
- May lead to anemic domain model

**When to use:**
- Traditional web applications
- CRUD applications
- Enterprise systems
- When team is organized by technical layers

## Model-View-Controller (MVC)

### Overview

Separates application into three interconnected components.

```
     ┌─────────┐
     │  Model  │ (Data & Logic)
     └────┬────┘
          │
     ┌────┴────┐
     │         │
┌────▼───┐ ┌──▼──────┐
│  View  │ │Controller│
└────────┘ └─────────┘
```

### Example

```typescript
// Model
class UserModel {
  private users: User[] = [];

  getUser(id: string): User | undefined {
    return this.users.find(u => u.id === id);
  }

  addUser(user: User): void {
    this.users.push(user);
  }

  updateUser(id: string, data: Partial<User>): void {
    const user = this.getUser(id);
    if (user) Object.assign(user, data);
  }
}

// View (React example)
const UserView = ({ user, onEdit }: UserViewProps) => {
  return (
    <div className="user-profile">
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <button onClick={onEdit}>Edit</button>
    </div>
  );
};

// Controller
class UserController {
  constructor(
    private model: UserModel,
    private view: typeof UserView
  ) {}

  getUser(id: string) {
    const user = this.model.getUser(id);
    return this.view({ user, onEdit: () => this.editUser(id) });
  }

  editUser(id: string) {
    // Handle edit logic
  }
}
```

**Pros:**
- Clear separation of concerns
- Easier to test
- Multiple views for same model
- Familiar pattern

**Cons:**
- Can be overkill for simple apps
- Tight coupling between components
- Complex for large applications

**When to use:**
- Web applications
- Desktop applications
- When you need multiple views
- Traditional server-rendered apps

## Model-View-ViewModel (MVVM)

### Overview

Separates UI from business logic with ViewModel as intermediary.

```
┌───────┐         ┌──────────┐         ┌───────┐
│ View  │◄────────┤ ViewModel│◄────────┤ Model │
└───────┘  Binding└──────────┘  Data   └───────┘
```

### Example

```typescript
// Model
interface User {
  id: string;
  name: string;
  email: string;
}

// ViewModel
class UserViewModel {
  @observable user: User | null = null;
  @observable loading: boolean = false;
  @observable error: string | null = null;

  constructor(private userService: UserService) {
    makeObservable(this);
  }

  @action
  async loadUser(id: string) {
    this.loading = true;
    try {
      this.user = await this.userService.getUser(id);
      this.error = null;
    } catch (err) {
      this.error = err.message;
    } finally {
      this.loading = false;
    }
  }

  @computed
  get displayName(): string {
    return this.user?.name || 'Guest';
  }

  @action
  updateName(name: string) {
    if (this.user) {
      this.user.name = name;
    }
  }
}

// View
const UserProfile = observer(({ viewModel }: { viewModel: UserViewModel }) => {
  if (viewModel.loading) return <Spinner />;
  if (viewModel.error) return <Error message={viewModel.error} />;

  return (
    <div>
      <h1>{viewModel.displayName}</h1>
      <input
        value={viewModel.user?.name}
        onChange={(e) => viewModel.updateName(e.target.value)}
      />
    </div>
  );
});
```

**Pros:**
- Strong data binding
- Testable without UI
- Clean separation
- Reactive updates

**Cons:**
- Learning curve
- Can be complex
- Requires framework support

**When to use:**
- Angular, WPF, Xamarin apps
- Complex UIs with lots of state
- When you want strong data binding

## Microservices Architecture

### Overview

Application as collection of small, independent services.

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│ User     │  │ Product  │  │ Order    │
│ Service  │  │ Service  │  │ Service  │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │             │
     └─────────┬───┴─────────────┘
               │
         ┌─────▼─────┐
         │  API      │
         │  Gateway  │
         └───────────┘
```

### Example

```typescript
// User Service
class UserService {
  private port = 3001;

  start() {
    app.get('/users/:id', async (req, res) => {
      const user = await userRepository.findById(req.params.id);
      res.json(user);
    });

    app.listen(this.port);
  }
}

// Order Service
class OrderService {
  private port = 3002;
  private userServiceUrl = 'http://user-service:3001';

  async createOrder(orderData: OrderData) {
    // Call User Service
    const user = await fetch(`${this.userServiceUrl}/users/${orderData.userId}`)
      .then(r => r.json());

    // Create order
    const order = await orderRepository.create({
      ...orderData,
      userName: user.name,
    });

    return order;
  }

  start() {
    app.post('/orders', async (req, res) => {
      const order = await this.createOrder(req.body);
      res.json(order);
    });

    app.listen(this.port);
  }
}

// API Gateway
class APIGateway {
  routes = {
    '/api/users': 'http://user-service:3001',
    '/api/orders': 'http://order-service:3002',
    '/api/products': 'http://product-service:3003',
  };

  start() {
    app.use('/api/*', (req, res) => {
      const service = this.getService(req.path);
      // Proxy request to appropriate service
    });
  }
}
```

**Pros:**
- Independent deployment
- Technology flexibility
- Scalability
- Team autonomy
- Fault isolation

**Cons:**
- Complex deployment
- Distributed system challenges
- Data consistency issues
- Operational overhead
- Network latency

**When to use:**
- Large, complex applications
- Multiple teams
- Need independent scaling
- Different tech stacks

## Event-Driven Architecture

### Overview

Components communicate through events.

```
┌──────────┐    Event    ┌──────────┐
│ Producer │────────────►│Event Bus │
└──────────┘             └────┬─────┘
                              │
              ┌───────────────┼───────────────┐
              │               │               │
         ┌────▼────┐    ┌────▼────┐    ┌────▼────┐
         │Consumer1│    │Consumer2│    │Consumer3│
         └─────────┘    └─────────┘    └─────────┘
```

### Example

```typescript
// Event Bus
class EventBus {
  private listeners = new Map<string, Set<Function>>();

  on(event: string, handler: Function) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(handler);
  }

  emit(event: string, data: any) {
    this.listeners.get(event)?.forEach(handler => handler(data));
  }
}

// Producer
class OrderService {
  constructor(private eventBus: EventBus) {}

  async createOrder(orderData: OrderData) {
    const order = await this.saveOrder(orderData);

    // Emit event
    this.eventBus.emit('order.created', {
      orderId: order.id,
      userId: order.userId,
      total: order.total,
      timestamp: new Date(),
    });

    return order;
  }
}

// Consumer 1 - Send Email
class EmailService {
  constructor(private eventBus: EventBus) {
    this.eventBus.on('order.created', this.handleOrderCreated.bind(this));
  }

  private async handleOrderCreated(event: OrderCreatedEvent) {
    await this.sendEmail({
      to: event.userEmail,
      subject: 'Order Confirmation',
      body: `Your order ${event.orderId} has been created.`,
    });
  }
}

// Consumer 2 - Update Inventory
class InventoryService {
  constructor(private eventBus: EventBus) {
    this.eventBus.on('order.created', this.handleOrderCreated.bind(this));
  }

  private async handleOrderCreated(event: OrderCreatedEvent) {
    await this.updateInventory(event.items);
  }
}

// Consumer 3 - Analytics
class AnalyticsService {
  constructor(private eventBus: EventBus) {
    this.eventBus.on('order.created', this.trackOrder.bind(this));
  }

  private async trackOrder(event: OrderCreatedEvent) {
    await this.track('order_created', {
      orderId: event.orderId,
      total: event.total,
    });
  }
}
```

**Pros:**
- Loose coupling
- Scalability
- Flexibility
- Easy to add new features
- Asynchronous processing

**Cons:**
- Complex debugging
- Event ordering issues
- Eventual consistency
- Difficult to trace flow

**When to use:**
- Complex workflows
- Multiple systems need to react
- Asynchronous processing
- Microservices communication

## Repository Pattern

### Overview

Abstracts data access logic.

```typescript
// Domain Model
interface User {
  id: string;
  name: string;
  email: string;
}

// Repository Interface
interface IUserRepository {
  findById(id: string): Promise<User | null>;
  findAll(): Promise<User[]>;
  create(user: User): Promise<User>;
  update(id: string, user: Partial<User>): Promise<User>;
  delete(id: string): Promise<void>;
}

// Implementation
class UserRepository implements IUserRepository {
  async findById(id: string): Promise<User | null> {
    return db.users.findOne({ id });
  }

  async findAll(): Promise<User[]> {
    return db.users.find({});
  }

  async create(user: User): Promise<User> {
    return db.users.insertOne(user);
  }

  async update(id: string, data: Partial<User>): Promise<User> {
    return db.users.updateOne({ id }, data);
  }

  async delete(id: string): Promise<void> {
    await db.users.deleteOne({ id });
  }
}

// Usage in Service
class UserService {
  constructor(private repository: IUserRepository) {}

  async getUser(id: string): Promise<User> {
    const user = await this.repository.findById(id);
    if (!user) throw new Error('User not found');
    return user;
  }
}

// Easy to test with mock
class MockUserRepository implements IUserRepository {
  private users: User[] = [];

  async findById(id: string): Promise<User | null> {
    return this.users.find(u => u.id === id) || null;
  }
  // ... other methods
}

// Test
const mockRepo = new MockUserRepository();
const userService = new UserService(mockRepo);
```

**Pros:**
- Abstracts data access
- Easy to test
- Flexible data sources
- Centralized data logic

**Cons:**
- Additional abstraction layer
- Can be overkill for simple apps

**When to use:**
- Complex data access
- Multiple data sources
- Need testability
- Domain-driven design

## CQRS (Command Query Responsibility Segregation)

### Overview

Separate read and write operations.

```
         ┌──────────┐
         │  Client  │
         └────┬─────┘
              │
       ┌──────┴──────┐
       │             │
  ┌────▼────┐   ┌───▼────┐
  │Commands │   │ Queries│
  │ (Write) │   │ (Read) │
  └────┬────┘   └───┬────┘
       │            │
  ┌────▼────┐   ┌───▼─────┐
  │ Write   │   │  Read   │
  │   DB    │   │   DB    │
  └─────────┘   └─────────┘
```

### Example

```typescript
// Command Side (Write)
interface CreateUserCommand {
  name: string;
  email: string;
}

class UserCommandHandler {
  async handle(command: CreateUserCommand): Promise<void> {
    const user = new User(command.name, command.email);
    await writeDB.users.insert(user);

    // Publish event for read side
    eventBus.emit('user.created', user);
  }
}

// Query Side (Read)
interface UserQuery {
  id: string;
}

class UserQueryHandler {
  async handle(query: UserQuery): Promise<UserDTO> {
    return readDB.users.findById(query.id);
  }
}

// Separate read models optimized for queries
interface UserListDTO {
  id: string;
  name: string;
  email: string;
  orderCount: number;
  lastOrderDate: Date;
}
```

**Pros:**
- Optimized reads and writes
- Better scalability
- Flexible data models
- Clear intent

**Cons:**
- Increased complexity
- Eventual consistency
- More infrastructure

**When to use:**
- High read/write ratio difference
- Complex domain logic
- Need different scalability for reads/writes

## Further Reading

- [Frontend Architecture Patterns](../frontend/fundamentals.md)
- [Design Principles](./principles.md)
- [System Design](./system-design.md)

---

*Next: [System Design →](./system-design.md)*

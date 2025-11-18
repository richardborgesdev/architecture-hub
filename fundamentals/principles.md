# Architecture Principles

## Introduction

Software architecture principles are fundamental guidelines that shape how we design and build systems. These principles have stood the test of time and apply across different domains and technologies.

## SOLID Principles

### S - Single Responsibility Principle (SRP)

**"A class should have one, and only one, reason to change."**

```typescript
// ❌ Bad: Multiple responsibilities
class UserManager {
  createUser(data: UserData) { /* ... */ }
  sendEmail(email: string) { /* ... */ }
  logActivity(activity: string) { /* ... */ }
  validateUser(user: User) { /* ... */ }
}

// ✅ Good: Single responsibility
class UserService {
  createUser(data: UserData) { /* ... */ }
}

class EmailService {
  sendEmail(email: string) { /* ... */ }
}

class ActivityLogger {
  logActivity(activity: string) { /* ... */ }
}

class UserValidator {
  validate(user: User) { /* ... */ }
}
```

**Benefits:**
- Easier to understand
- Simpler to test
- Easier to maintain
- More reusable

### O - Open/Closed Principle (OCP)

**"Software entities should be open for extension but closed for modification."**

```typescript
// ❌ Bad: Must modify to add new payment methods
class PaymentProcessor {
  process(payment: Payment) {
    if (payment.type === 'credit-card') {
      // Process credit card
    } else if (payment.type === 'paypal') {
      // Process PayPal
    } else if (payment.type === 'crypto') {
      // Process crypto
    }
  }
}

// ✅ Good: Open for extension, closed for modification
interface PaymentMethod {
  process(payment: Payment): Promise<PaymentResult>;
}

class CreditCardPayment implements PaymentMethod {
  async process(payment: Payment) {
    // Process credit card
  }
}

class PayPalPayment implements PaymentMethod {
  async process(payment: Payment) {
    // Process PayPal
  }
}

class PaymentProcessor {
  constructor(private methods: Map<string, PaymentMethod>) {}

  async process(payment: Payment) {
    const method = this.methods.get(payment.type);
    if (!method) throw new Error('Unsupported payment method');
    return method.process(payment);
  }
}
```

### L - Liskov Substitution Principle (LSP)

**"Derived classes must be substitutable for their base classes."**

```typescript
// ❌ Bad: Square violates LSP
class Rectangle {
  width: number;
  height: number;

  setWidth(width: number) { this.width = width; }
  setHeight(height: number) { this.height = height; }

  getArea() { return this.width * this.height; }
}

class Square extends Rectangle {
  setWidth(width: number) {
    this.width = width;
    this.height = width; // Violates LSP - unexpected behavior
  }

  setHeight(height: number) {
    this.width = height;
    this.height = height;
  }
}

// ✅ Good: Use composition instead
interface Shape {
  getArea(): number;
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}

  getArea() {
    return this.width * this.height;
  }
}

class Square implements Shape {
  constructor(private size: number) {}

  getArea() {
    return this.size * this.size;
  }
}
```

### I - Interface Segregation Principle (ISP)

**"Clients should not be forced to depend on interfaces they don't use."**

```typescript
// ❌ Bad: Fat interface
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
}

class Human implements Worker {
  work() { /* ... */ }
  eat() { /* ... */ }
  sleep() { /* ... */ }
}

class Robot implements Worker {
  work() { /* ... */ }
  eat() { throw new Error('Robots don\'t eat'); } // Forced to implement
  sleep() { throw new Error('Robots don\'t sleep'); }
}

// ✅ Good: Segregated interfaces
interface Workable {
  work(): void;
}

interface Eatable {
  eat(): void;
}

interface Sleepable {
  sleep(): void;
}

class Human implements Workable, Eatable, Sleepable {
  work() { /* ... */ }
  eat() { /* ... */ }
  sleep() { /* ... */ }
}

class Robot implements Workable {
  work() { /* ... */ }
}
```

### D - Dependency Inversion Principle (DIP)

**"Depend on abstractions, not concretions."**

```typescript
// ❌ Bad: High-level module depends on low-level module
class MySQLDatabase {
  save(data: any) { /* ... */ }
}

class UserService {
  private db = new MySQLDatabase(); // Direct dependency

  saveUser(user: User) {
    this.db.save(user);
  }
}

// ✅ Good: Both depend on abstraction
interface Database {
  save(data: any): Promise<void>;
  find(id: string): Promise<any>;
}

class MySQLDatabase implements Database {
  async save(data: any) { /* ... */ }
  async find(id: string) { /* ... */ }
}

class PostgreSQLDatabase implements Database {
  async save(data: any) { /* ... */ }
  async find(id: string) { /* ... */ }
}

class UserService {
  constructor(private db: Database) {} // Depends on abstraction

  async saveUser(user: User) {
    await this.db.save(user);
  }
}

// Usage with dependency injection
const db = new MySQLDatabase();
const userService = new UserService(db);
```

## Other Key Principles

### DRY (Don't Repeat Yourself)

**"Every piece of knowledge must have a single, unambiguous representation."**

```typescript
// ❌ Bad: Repeated logic
function getUserFullName(user: User) {
  return `${user.firstName} ${user.lastName}`;
}

function displayUserName(user: User) {
  return `${user.firstName} ${user.lastName}`; // Duplication
}

// ✅ Good: Single source of truth
class User {
  constructor(
    public firstName: string,
    public lastName: string
  ) {}

  getFullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}

function displayUserName(user: User) {
  return user.getFullName();
}
```

### KISS (Keep It Simple, Stupid)

**"Simplicity should be a key goal in design."**

```typescript
// ❌ Bad: Over-engineered
class NumberProcessor {
  private strategy: ProcessingStrategy;
  private validator: NumberValidator;
  private transformer: NumberTransformer;

  process(num: number) {
    if (!this.validator.validate(num)) {
      throw new Error('Invalid');
    }
    const transformed = this.transformer.transform(num);
    return this.strategy.execute(transformed);
  }
}

// ✅ Good: Simple and clear
function isEven(num: number): boolean {
  return num % 2 === 0;
}
```

### YAGNI (You Aren't Gonna Need It)

**"Don't add functionality until you need it."**

```typescript
// ❌ Bad: Building for hypothetical future needs
class User {
  id: string;
  name: string;
  email: string;
  // Future-proofing that's not needed yet
  alternateEmails?: string[];
  preferences?: Map<string, any>;
  metadata?: Record<string, unknown>;
  tags?: string[];
  customFields?: any[];
}

// ✅ Good: Only what's needed now
class User {
  id: string;
  name: string;
  email: string;
}
// Add more fields when actually needed
```

### Separation of Concerns (SoC)

**"Separate a program into distinct sections, each addressing a separate concern."**

```typescript
// ❌ Bad: Mixed concerns
const UserProfile = () => {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch('/api/user')
      .then(res => res.json())
      .then(data => {
        // Validation logic
        if (!data.email.includes('@')) {
          throw new Error('Invalid email');
        }
        // Transformation logic
        const formatted = {
          ...data,
          fullName: `${data.firstName} ${data.lastName}`,
        };
        setUser(formatted);
      });
  }, []);

  return <div>{user?.fullName}</div>;
};

// ✅ Good: Separated concerns
// Data fetching
const useUser = () => {
  return useQuery(['user'], fetchUser);
};

// Validation
const validateUser = (user: User) => {
  if (!user.email.includes('@')) {
    throw new Error('Invalid email');
  }
};

// Transformation
const formatUser = (user: User) => ({
  ...user,
  fullName: `${user.firstName} ${user.lastName}`,
});

// Presentation
const UserProfile = () => {
  const { data: user } = useUser();

  if (!user) return <Loading />;

  const formattedUser = formatUser(user);
  return <div>{formattedUser.fullName}</div>;
};
```

### Composition Over Inheritance

**"Prefer object composition to class inheritance."**

```typescript
// ❌ Bad: Deep inheritance hierarchy
class Animal {
  eat() { /* ... */ }
}

class Mammal extends Animal {
  breathe() { /* ... */ }
}

class Dog extends Mammal {
  bark() { /* ... */ }
}

class Cat extends Mammal {
  meow() { /* ... */ }
}

// ✅ Good: Composition
interface Eatable {
  eat(): void;
}

interface Breathable {
  breathe(): void;
}

interface Vocalizable {
  makeSound(): void;
}

class Dog implements Eatable, Breathable, Vocalizable {
  eat() { /* ... */ }
  breathe() { /* ... */ }
  makeSound() { console.log('Woof!'); }
}

// Or with composition
class Dog {
  constructor(
    private eating: Eatable,
    private breathing: Breathable,
    private vocalization: Vocalizable
  ) {}

  eat() { this.eating.eat(); }
  breathe() { this.breathing.breathe(); }
  makeSound() { this.vocalization.makeSound(); }
}
```

### Law of Demeter (Principle of Least Knowledge)

**"A unit should have only limited knowledge about other units."**

```typescript
// ❌ Bad: Too much knowledge about structure
class OrderProcessor {
  process(order: Order) {
    const street = order.customer.address.street; // Chain of calls
    const city = order.customer.address.city;
    // Process...
  }
}

// ✅ Good: Tell, don't ask
class Order {
  getShippingAddress(): string {
    return this.customer.getFullAddress();
  }
}

class Customer {
  getFullAddress(): string {
    return this.address.format();
  }
}

class OrderProcessor {
  process(order: Order) {
    const address = order.getShippingAddress();
    // Process...
  }
}
```

### Convention Over Configuration

**"Decrease the number of decisions developers need to make."**

```typescript
// ✅ Use sensible defaults
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
}

const Button = ({
  variant = 'primary',  // Convention: primary by default
  size = 'md',          // Convention: medium by default
  disabled = false,     // Convention: enabled by default
  ...props
}: ButtonProps) => {
  // No need to configure everything every time
};
```

### Fail Fast

**"Report problems immediately and visibly."**

```typescript
// ✅ Validate early
function processPayment(amount: number, currency: string) {
  // Fail fast - validate inputs immediately
  if (amount <= 0) {
    throw new Error('Amount must be positive');
  }

  if (!['USD', 'EUR', 'GBP'].includes(currency)) {
    throw new Error('Unsupported currency');
  }

  // Continue with processing...
}

// ✅ Use TypeScript for compile-time checks
type Currency = 'USD' | 'EUR' | 'GBP'; // Fails at compile time

function processPayment(amount: number, currency: Currency) {
  // Type system ensures valid currency
}
```

### Principle of Least Astonishment

**"Design should match user expectations."**

```typescript
// ❌ Bad: Surprising behavior
const deleteUser = (userId: string) => {
  // Surprisingly doesn't delete, just marks as inactive
  return updateUser(userId, { active: false });
};

// ✅ Good: Does what name suggests
const deleteUser = (userId: string) => {
  return api.delete(`/users/${userId}`);
};

const deactivateUser = (userId: string) => {
  return updateUser(userId, { active: false });
};
```

### Encapsulation

**"Hide internal state and implementation details."**

```typescript
// ❌ Bad: Exposing internals
class BankAccount {
  public balance: number = 0;

  // Anyone can modify balance directly
}

const account = new BankAccount();
account.balance = 1000000; // Direct manipulation

// ✅ Good: Controlled access
class BankAccount {
  private balance: number = 0;

  deposit(amount: number) {
    if (amount <= 0) throw new Error('Invalid amount');
    this.balance += amount;
  }

  withdraw(amount: number) {
    if (amount > this.balance) throw new Error('Insufficient funds');
    this.balance -= amount;
  }

  getBalance(): number {
    return this.balance;
  }
}
```

## Architectural Qualities

### Maintainability

Code is easy to change and extend:

```typescript
// ✅ Well-structured, maintainable code
// Clear structure
// Single responsibility
// Well-documented
// Tested
```

### Scalability

System handles growth:

```typescript
// Horizontal scaling: Add more instances
// Vertical scaling: Increase resources
// Code splitting: Load on demand
// Caching: Reduce redundant work
```

### Testability

Easy to write tests:

```typescript
// ✅ Testable: Pure functions, dependency injection
function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// ❌ Hard to test: Side effects, hidden dependencies
function calculateTotal() {
  const items = fetchItemsFromDatabase();
  sendAnalyticsEvent('total_calculated');
  return items.reduce((sum, item) => sum + item.price, 0);
}
```

### Performance

Efficient resource usage:

```typescript
// Lazy loading
// Memoization
// Caching
// Code splitting
// Optimistic updates
```

### Security

Protected from vulnerabilities:

```typescript
// Input validation
// Authentication/Authorization
// Encryption
// Secure communication
// Principle of least privilege
```

## Applying Principles

### Balance is Key

Principles are guidelines, not rules:

```typescript
// Don't over-apply principles
// Sometimes a simple if/else is better than a strategy pattern
// YAGNI applies to architecture too

// Example: Simple is fine for small apps
const getDiscount = (type: string) => {
  if (type === 'student') return 0.2;
  if (type === 'senior') return 0.15;
  return 0;
};

// Only create abstractions when complexity justifies it
```

### Context Matters

Choose principles based on:
- Team size and experience
- Application complexity
- Performance requirements
- Time constraints
- Business needs

## Best Practices

### ✅ Do

- Understand the principle before applying
- Apply principles that solve actual problems
- Review and refactor regularly
- Document architectural decisions
- Consider trade-offs
- Keep it pragmatic

### ❌ Don't

- Follow principles blindly
- Over-engineer solutions
- Apply all principles everywhere
- Ignore business context
- Create abstractions prematurely
- Sacrifice readability for principles

## Further Reading

- [Architecture Patterns](./patterns.md)
- [System Design](./system-design.md)
- [Architecture Decision Records](./adr.md)

---

*Next: [Architecture Patterns →](./patterns.md)*

# Creational Design Patterns

## Introduction

Creational patterns deal with object creation mechanisms, providing flexibility in what gets created, who creates it, how it gets created, and when. These patterns help make systems independent of how objects are created, composed, and represented.

## Singleton Pattern

### Purpose
Ensure a class has only one instance and provide a global point of access to it.

### Implementation

```typescript
class DatabaseConnection {
  private static instance: DatabaseConnection;
  private connection: any;

  // Private constructor prevents direct instantiation
  private constructor() {
    this.connection = this.createConnection();
  }

  public static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }
    return DatabaseConnection.instance;
  }

  private createConnection() {
    console.log('Creating database connection...');
    return { /* connection object */ };
  }

  public query(sql: string) {
    return this.connection.execute(sql);
  }
}

// Usage
const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();
console.log(db1 === db2); // true - same instance
```

### Modern Approach (ES6 Modules)

```typescript
// database.ts
class DatabaseConnection {
  private connection: any;

  constructor() {
    this.connection = this.createConnection();
  }

  private createConnection() {
    return { /* connection */ };
  }

  query(sql: string) {
    return this.connection.execute(sql);
  }
}

// Export a single instance
export const db = new DatabaseConnection();

// usage.ts
import { db } from './database';
db.query('SELECT * FROM users');
```

**When to use:**
- Database connections
- Configuration objects
- Logging services
- Cache managers

**Cautions:**
- Can make testing difficult
- Global state concerns
- Consider dependency injection instead

## Factory Pattern

### Purpose
Create objects without specifying exact class.

### Implementation

```typescript
// Product interface
interface Button {
  render(): void;
  onClick(): void;
}

// Concrete products
class IOSButton implements Button {
  render() {
    console.log('Rendering iOS button');
  }

  onClick() {
    console.log('iOS button clicked');
  }
}

class AndroidButton implements Button {
  render() {
    console.log('Rendering Android button');
  }

  onClick() {
    console.log('Android button clicked');
  }
}

class WebButton implements Button {
  render() {
    console.log('Rendering Web button');
  }

  onClick() {
    console.log('Web button clicked');
  }
}

// Factory
class ButtonFactory {
  static createButton(type: 'ios' | 'android' | 'web'): Button {
    switch (type) {
      case 'ios':
        return new IOSButton();
      case 'android':
        return new AndroidButton();
      case 'web':
        return new WebButton();
      default:
        throw new Error(`Unknown button type: ${type}`);
    }
  }
}

// Usage
const platform = detectPlatform(); // 'ios' | 'android' | 'web'
const button = ButtonFactory.createButton(platform);
button.render();
```

### Factory with Parameters

```typescript
interface User {
  type: string;
  permissions: string[];
}

class AdminUser implements User {
  type = 'admin';
  permissions = ['read', 'write', 'delete', 'admin'];
}

class RegularUser implements User {
  type = 'user';
  permissions = ['read', 'write'];
}

class GuestUser implements User {
  type = 'guest';
  permissions = ['read'];
}

class UserFactory {
  static createUser(userType: string): User {
    switch (userType) {
      case 'admin':
        return new AdminUser();
      case 'user':
        return new RegularUser();
      case 'guest':
      default:
        return new GuestUser();
    }
  }
}

// Usage
const user = UserFactory.createUser('admin');
console.log(user.permissions); // ['read', 'write', 'delete', 'admin']
```

**When to use:**
- Object creation is complex
- Need to decouple creation from usage
- Creating different types based on conditions
- Plugin systems

## Abstract Factory Pattern

### Purpose
Create families of related objects without specifying concrete classes.

### Implementation

```typescript
// Abstract products
interface Button {
  render(): void;
}

interface Checkbox {
  render(): void;
}

// Concrete products for Windows
class WindowsButton implements Button {
  render() {
    console.log('Render Windows button');
  }
}

class WindowsCheckbox implements Checkbox {
  render() {
    console.log('Render Windows checkbox');
  }
}

// Concrete products for Mac
class MacButton implements Button {
  render() {
    console.log('Render Mac button');
  }
}

class MacCheckbox implements Checkbox {
  render() {
    console.log('Render Mac checkbox');
  }
}

// Abstract factory
interface GUIFactory {
  createButton(): Button;
  createCheckbox(): Checkbox;
}

// Concrete factories
class WindowsFactory implements GUIFactory {
  createButton(): Button {
    return new WindowsButton();
  }

  createCheckbox(): Checkbox {
    return new WindowsCheckbox();
  }
}

class MacFactory implements GUIFactory {
  createButton(): Button {
    return new MacButton();
  }

  createCheckbox(): Checkbox {
    return new MacCheckbox();
  }
}

// Client code
class Application {
  private button: Button;
  private checkbox: Checkbox;

  constructor(factory: GUIFactory) {
    this.button = factory.createButton();
    this.checkbox = factory.createCheckbox();
  }

  render() {
    this.button.render();
    this.checkbox.render();
  }
}

// Usage
const os = detectOS(); // 'windows' | 'mac'
const factory = os === 'windows' ? new WindowsFactory() : new MacFactory();
const app = new Application(factory);
app.render();
```

**When to use:**
- System needs to be independent of product creation
- System configured with multiple product families
- Related products should be used together
- UI component libraries for different platforms

## Builder Pattern

### Purpose
Construct complex objects step by step.

### Implementation

```typescript
// Product
class Pizza {
  size?: string;
  cheese?: boolean;
  pepperoni?: boolean;
  mushrooms?: boolean;
  olives?: boolean;

  describe() {
    console.log(`${this.size} pizza with:`);
    if (this.cheese) console.log('- Cheese');
    if (this.pepperoni) console.log('- Pepperoni');
    if (this.mushrooms) console.log('- Mushrooms');
    if (this.olives) console.log('- Olives');
  }
}

// Builder
class PizzaBuilder {
  private pizza: Pizza;

  constructor() {
    this.pizza = new Pizza();
  }

  setSize(size: 'small' | 'medium' | 'large'): this {
    this.pizza.size = size;
    return this;
  }

  addCheese(): this {
    this.pizza.cheese = true;
    return this;
  }

  addPepperoni(): this {
    this.pizza.pepperoni = true;
    return this;
  }

  addMushrooms(): this {
    this.pizza.mushrooms = true;
    return this;
  }

  addOlives(): this {
    this.pizza.olives = true;
    return this;
  }

  build(): Pizza {
    return this.pizza;
  }
}

// Usage
const pizza = new PizzaBuilder()
  .setSize('large')
  .addCheese()
  .addPepperoni()
  .addMushrooms()
  .build();

pizza.describe();
```

### Fluent API Builder

```typescript
class QueryBuilder {
  private query = {
    select: [] as string[],
    from: '',
    where: [] as string[],
    orderBy: [] as string[],
  };

  select(...fields: string[]): this {
    this.query.select.push(...fields);
    return this;
  }

  from(table: string): this {
    this.query.from = table;
    return this;
  }

  where(condition: string): this {
    this.query.where.push(condition);
    return this;
  }

  orderBy(field: string, direction: 'ASC' | 'DESC' = 'ASC'): this {
    this.query.orderBy.push(`${field} ${direction}`);
    return this;
  }

  build(): string {
    let sql = `SELECT ${this.query.select.join(', ')}`;
    sql += ` FROM ${this.query.from}`;

    if (this.query.where.length > 0) {
      sql += ` WHERE ${this.query.where.join(' AND ')}`;
    }

    if (this.query.orderBy.length > 0) {
      sql += ` ORDER BY ${this.query.orderBy.join(', ')}`;
    }

    return sql;
  }
}

// Usage
const sql = new QueryBuilder()
  .select('id', 'name', 'email')
  .from('users')
  .where('age > 18')
  .where('active = true')
  .orderBy('name', 'ASC')
  .build();

console.log(sql);
// SELECT id, name, email FROM users WHERE age > 18 AND active = true ORDER BY name ASC
```

**When to use:**
- Complex object construction
- Many optional parameters
- Step-by-step creation process
- Immutable objects

## Prototype Pattern

### Purpose
Create new objects by copying existing objects.

### Implementation

```typescript
interface Cloneable<T> {
  clone(): T;
}

class Shape implements Cloneable<Shape> {
  constructor(
    public x: number,
    public y: number,
    public color: string
  ) {}

  clone(): Shape {
    return new Shape(this.x, this.y, this.color);
  }
}

class Circle extends Shape {
  constructor(
    x: number,
    y: number,
    color: string,
    public radius: number
  ) {
    super(x, y, color);
  }

  clone(): Circle {
    return new Circle(this.x, this.y, this.color, this.radius);
  }
}

// Usage
const original = new Circle(10, 20, 'red', 15);
const copy = original.clone();

copy.x = 50;
copy.color = 'blue';

console.log(original.x); // 10
console.log(copy.x);     // 50
```

### Deep Clone

```typescript
class Node {
  constructor(
    public value: number,
    public children: Node[] = []
  ) {}

  clone(): Node {
    // Deep clone children
    const clonedChildren = this.children.map(child => child.clone());
    return new Node(this.value, clonedChildren);
  }
}

// Usage
const tree = new Node(1, [
  new Node(2),
  new Node(3, [
    new Node(4),
    new Node(5)
  ])
]);

const clonedTree = tree.clone();
clonedTree.children[0].value = 99;

console.log(tree.children[0].value);       // 2 (unchanged)
console.log(clonedTree.children[0].value); // 99 (changed)
```

**When to use:**
- Object creation is expensive
- Many similar objects needed
- Avoid subclasses for object creation
- Runtime object composition

## Object Pool Pattern

### Purpose
Reuse objects that are expensive to create.

### Implementation

```typescript
class DatabaseConnection {
  private inUse = false;

  constructor(private id: number) {
    console.log(`Creating connection ${id}`);
  }

  query(sql: string) {
    console.log(`Connection ${this.id}: ${sql}`);
  }

  setInUse(value: boolean) {
    this.inUse = value;
  }

  isInUse(): boolean {
    return this.inUse;
  }
}

class ConnectionPool {
  private static instance: ConnectionPool;
  private pool: DatabaseConnection[] = [];
  private maxSize = 5;

  private constructor() {
    // Initialize pool
    for (let i = 0; i < this.maxSize; i++) {
      this.pool.push(new DatabaseConnection(i));
    }
  }

  static getInstance(): ConnectionPool {
    if (!ConnectionPool.instance) {
      ConnectionPool.instance = new ConnectionPool();
    }
    return ConnectionPool.instance;
  }

  acquire(): DatabaseConnection {
    const connection = this.pool.find(conn => !conn.isInUse());

    if (!connection) {
      throw new Error('No connections available');
    }

    connection.setInUse(true);
    console.log('Connection acquired');
    return connection;
  }

  release(connection: DatabaseConnection) {
    connection.setInUse(false);
    console.log('Connection released');
  }
}

// Usage
const pool = ConnectionPool.getInstance();

const conn1 = pool.acquire();
conn1.query('SELECT * FROM users');

const conn2 = pool.acquire();
conn2.query('SELECT * FROM orders');

pool.release(conn1);

const conn3 = pool.acquire(); // Reuses conn1
conn3.query('SELECT * FROM products');
```

**When to use:**
- Object creation is expensive
- Many short-lived objects
- Limited resources (connections, threads)
- Performance optimization needed

## Comparison

| Pattern | Purpose | When to Use |
|---------|---------|-------------|
| **Singleton** | One instance only | Shared resource (config, logger) |
| **Factory** | Create objects without specifying class | Plugin systems, conditional creation |
| **Abstract Factory** | Families of related objects | Multiple product families |
| **Builder** | Complex step-by-step construction | Many optional parameters |
| **Prototype** | Clone existing objects | Expensive object creation |
| **Object Pool** | Reuse expensive objects | Limited resources, performance |

## Best Practices

### ✅ Do

- Use factories for complex creation logic
- Use builders for objects with many parameters
- Pool expensive resources
- Consider prototype for performance
- Document singleton usage

### ❌ Don't

- Overuse singletons (global state)
- Create factories for simple objects
- Clone objects with circular references carelessly
- Forget to release pooled resources
- Make builders too complex

## Further Reading

- [Structural Patterns](./structural.md)
- [Behavioral Patterns](./behavioral.md)
- [Frontend Patterns](./frontend-patterns.md)

---

*Next: [Structural Patterns →](./structural.md)*

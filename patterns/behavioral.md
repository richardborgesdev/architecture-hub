# Behavioral Design Patterns

## Introduction

Behavioral patterns are concerned with algorithms and the assignment of responsibilities between objects. They describe not just patterns of objects or classes but also the patterns of communication between them.

## Observer Pattern

### Purpose
Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified automatically.

### Implementation

```typescript
// Subject interface
interface Subject {
  attach(observer: Observer): void;
  detach(observer: Observer): void;
  notify(): void;
}

// Observer interface
interface Observer {
  update(subject: Subject): void;
}

// Concrete Subject
class ConcreteSubject implements Subject {
  private observers: Observer[] = [];
  private state: number = 0;

  attach(observer: Observer) {
    const isExist = this.observers.includes(observer);
    if (isExist) {
      console.log('Observer already attached');
      return;
    }
    this.observers.push(observer);
    console.log('Observer attached');
  }

  detach(observer: Observer) {
    const observerIndex = this.observers.indexOf(observer);
    if (observerIndex === -1) {
      console.log('Observer not found');
      return;
    }
    this.observers.splice(observerIndex, 1);
    console.log('Observer detached');
  }

  notify() {
    console.log('Notifying observers...');
    for (const observer of this.observers) {
      observer.update(this);
    }
  }

  setState(state: number) {
    console.log(`State changed to: ${state}`);
    this.state = state;
    this.notify();
  }

  getState(): number {
    return this.state;
  }
}

// Concrete Observers
class ConcreteObserverA implements Observer {
  update(subject: Subject) {
    if (subject instanceof ConcreteSubject && subject.getState() < 3) {
      console.log('ObserverA: Reacted to the event');
    }
  }
}

class ConcreteObserverB implements Observer {
  update(subject: Subject) {
    if (subject instanceof ConcreteSubject && subject.getState() >= 3) {
      console.log('ObserverB: Reacted to the event');
    }
  }
}

// Usage
const subject = new ConcreteSubject();
const observer1 = new ConcreteObserverA();
const observer2 = new ConcreteObserverB();

subject.attach(observer1);
subject.attach(observer2);

subject.setState(2); // ObserverA reacts
subject.setState(5); // ObserverB reacts
```

### Real-World Example: Event System

```typescript
type EventCallback = (...args: any[]) => void;

class EventEmitter {
  private events: Map<string, EventCallback[]> = new Map();

  on(event: string, callback: EventCallback) {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event)!.push(callback);
  }

  off(event: string, callback: EventCallback) {
    const callbacks = this.events.get(event);
    if (callbacks) {
      const index = callbacks.indexOf(callback);
      if (index > -1) {
        callbacks.splice(index, 1);
      }
    }
  }

  emit(event: string, ...args: any[]) {
    const callbacks = this.events.get(event);
    if (callbacks) {
      callbacks.forEach(callback => callback(...args));
    }
  }

  once(event: string, callback: EventCallback) {
    const onceCallback = (...args: any[]) => {
      callback(...args);
      this.off(event, onceCallback);
    };
    this.on(event, onceCallback);
  }
}

// Usage
const emitter = new EventEmitter();

emitter.on('user:login', (user) => {
  console.log(`User logged in: ${user.name}`);
});

emitter.on('user:login', (user) => {
  console.log(`Welcome back, ${user.name}!`);
});

emitter.emit('user:login', { name: 'John', id: 1 });
// User logged in: John
// Welcome back, John!
```

**When to use:**
- Event handling systems
- Reactive programming
- MVC/MVVM architectures
- Pub/Sub systems

## Strategy Pattern

### Purpose
Define a family of algorithms, encapsulate each one, and make them interchangeable.

### Implementation

```typescript
// Strategy interface
interface PaymentStrategy {
  pay(amount: number): void;
}

// Concrete strategies
class CreditCardStrategy implements PaymentStrategy {
  constructor(
    private cardNumber: string,
    private cvv: string
  ) {}

  pay(amount: number) {
    console.log(`Paid $${amount} with credit card ${this.cardNumber}`);
  }
}

class PayPalStrategy implements PaymentStrategy {
  constructor(private email: string) {}

  pay(amount: number) {
    console.log(`Paid $${amount} via PayPal (${this.email})`);
  }
}

class CryptoStrategy implements PaymentStrategy {
  constructor(private walletAddress: string) {}

  pay(amount: number) {
    console.log(`Paid $${amount} with crypto to ${this.walletAddress}`);
  }
}

// Context
class ShoppingCart {
  private items: { name: string; price: number }[] = [];
  private paymentStrategy?: PaymentStrategy;

  addItem(name: string, price: number) {
    this.items.push({ name, price });
  }

  setPaymentStrategy(strategy: PaymentStrategy) {
    this.paymentStrategy = strategy;
  }

  checkout() {
    const total = this.items.reduce((sum, item) => sum + item.price, 0);

    if (!this.paymentStrategy) {
      throw new Error('Payment strategy not set');
    }

    this.paymentStrategy.pay(total);
  }
}

// Usage
const cart = new ShoppingCart();
cart.addItem('Book', 20);
cart.addItem('Pen', 5);

cart.setPaymentStrategy(new CreditCardStrategy('1234-5678', '123'));
cart.checkout(); // Paid $25 with credit card

cart.setPaymentStrategy(new PayPalStrategy('user@example.com'));
cart.checkout(); // Paid $25 via PayPal
```

### Modern Example: Sorting Strategies

```typescript
interface SortStrategy<T> {
  sort(data: T[]): T[];
}

class QuickSort<T> implements SortStrategy<T> {
  sort(data: T[]): T[] {
    console.log('Using QuickSort');
    // QuickSort implementation
    return [...data].sort();
  }
}

class MergeSort<T> implements SortStrategy<T> {
  sort(data: T[]): T[] {
    console.log('Using MergeSort');
    // MergeSort implementation
    return [...data].sort();
  }
}

class BubbleSort<T> implements SortStrategy<T> {
  sort(data: T[]): T[] {
    console.log('Using BubbleSort');
    // BubbleSort implementation
    return [...data].sort();
  }
}

class Sorter<T> {
  constructor(private strategy: SortStrategy<T>) {}

  setStrategy(strategy: SortStrategy<T>) {
    this.strategy = strategy;
  }

  sort(data: T[]): T[] {
    return this.strategy.sort(data);
  }
}

// Usage
const data = [5, 2, 8, 1, 9];
const sorter = new Sorter(new QuickSort());

const sorted1 = sorter.sort(data);
sorter.setStrategy(new MergeSort());
const sorted2 = sorter.sort(data);
```

**When to use:**
- Multiple algorithms for a task
- Avoid conditional statements
- Runtime algorithm selection
- Isolate business logic

## Command Pattern

### Purpose
Encapsulate a request as an object, allowing parameterization and queuing of requests.

### Implementation

```typescript
// Receiver
class Light {
  turnOn() {
    console.log('Light is ON');
  }

  turnOff() {
    console.log('Light is OFF');
  }
}

// Command interface
interface Command {
  execute(): void;
  undo(): void;
}

// Concrete commands
class LightOnCommand implements Command {
  constructor(private light: Light) {}

  execute() {
    this.light.turnOn();
  }

  undo() {
    this.light.turnOff();
  }
}

class LightOffCommand implements Command {
  constructor(private light: Light) {}

  execute() {
    this.light.turnOff();
  }

  undo() {
    this.light.turnOn();
  }
}

// Invoker
class RemoteControl {
  private history: Command[] = [];

  submit(command: Command) {
    command.execute();
    this.history.push(command);
  }

  undo() {
    const command = this.history.pop();
    if (command) {
      command.undo();
    }
  }
}

// Usage
const light = new Light();
const remote = new RemoteControl();

remote.submit(new LightOnCommand(light));  // Light is ON
remote.submit(new LightOffCommand(light)); // Light is OFF
remote.undo();                              // Light is ON
```

### Advanced Example: Text Editor

```typescript
interface TextCommand {
  execute(): void;
  undo(): void;
}

class TextEditor {
  private content = '';

  getContent(): string {
    return this.content;
  }

  setContent(content: string) {
    this.content = content;
  }

  append(text: string) {
    this.content += text;
  }

  delete(length: number) {
    this.content = this.content.slice(0, -length);
  }
}

class InsertTextCommand implements TextCommand {
  private previousContent: string;

  constructor(
    private editor: TextEditor,
    private text: string
  ) {
    this.previousContent = editor.getContent();
  }

  execute() {
    this.editor.append(this.text);
  }

  undo() {
    this.editor.setContent(this.previousContent);
  }
}

class DeleteTextCommand implements TextCommand {
  private previousContent: string;

  constructor(
    private editor: TextEditor,
    private length: number
  ) {
    this.previousContent = editor.getContent();
  }

  execute() {
    this.editor.delete(this.length);
  }

  undo() {
    this.editor.setContent(this.previousContent);
  }
}

class CommandManager {
  private history: TextCommand[] = [];
  private current = 0;

  execute(command: TextCommand) {
    command.execute();
    this.history = this.history.slice(0, this.current);
    this.history.push(command);
    this.current++;
  }

  undo() {
    if (this.current > 0) {
      this.current--;
      this.history[this.current].undo();
    }
  }

  redo() {
    if (this.current < this.history.length) {
      this.history[this.current].execute();
      this.current++;
    }
  }
}

// Usage
const editor = new TextEditor();
const manager = new CommandManager();

manager.execute(new InsertTextCommand(editor, 'Hello '));
console.log(editor.getContent()); // "Hello "

manager.execute(new InsertTextCommand(editor, 'World'));
console.log(editor.getContent()); // "Hello World"

manager.undo();
console.log(editor.getContent()); // "Hello "

manager.redo();
console.log(editor.getContent()); // "Hello World"
```

**When to use:**
- Undo/redo functionality
- Queue operations
- Log changes
- Macro recording

## State Pattern

### Purpose
Allow an object to alter its behavior when its internal state changes.

### Implementation

```typescript
// State interface
interface State {
  insertCoin(context: VendingMachine): void;
  pressButton(context: VendingMachine): void;
  dispense(context: VendingMachine): void;
}

// Context
class VendingMachine {
  private state: State;
  private count: number;

  constructor(count: number) {
    this.count = count;
    this.state = new NoCoinState();
  }

  setState(state: State) {
    this.state = state;
  }

  getCount(): number {
    return this.count;
  }

  releaseProduct() {
    if (this.count > 0) {
      this.count--;
    }
  }

  insertCoin() {
    this.state.insertCoin(this);
  }

  pressButton() {
    this.state.pressButton(this);
  }

  dispense() {
    this.state.dispense(this);
  }
}

// Concrete states
class NoCoinState implements State {
  insertCoin(context: VendingMachine) {
    console.log('Coin inserted');
    context.setState(new HasCoinState());
  }

  pressButton(context: VendingMachine) {
    console.log('Insert coin first');
  }

  dispense(context: VendingMachine) {
    console.log('Insert coin first');
  }
}

class HasCoinState implements State {
  insertCoin(context: VendingMachine) {
    console.log('Coin already inserted');
  }

  pressButton(context: VendingMachine) {
    console.log('Button pressed');
    if (context.getCount() > 0) {
      context.setState(new DispensingState());
    } else {
      console.log('Out of stock');
      context.setState(new NoCoinState());
    }
  }

  dispense(context: VendingMachine) {
    console.log('Press button first');
  }
}

class DispensingState implements State {
  insertCoin(context: VendingMachine) {
    console.log('Please wait, dispensing...');
  }

  pressButton(context: VendingMachine) {
    console.log('Already dispensing');
  }

  dispense(context: VendingMachine) {
    console.log('Product dispensed');
    context.releaseProduct();
    context.setState(new NoCoinState());
  }
}

// Usage
const machine = new VendingMachine(2);
machine.insertCoin();    // Coin inserted
machine.pressButton();   // Button pressed
machine.dispense();      // Product dispensed
```

**When to use:**
- Object behavior depends on state
- Large conditional statements
- State transitions
- Workflow systems

## Iterator Pattern

### Purpose
Provide a way to access elements of a collection sequentially without exposing its underlying representation.

### Implementation

```typescript
interface Iterator<T> {
  next(): T;
  hasNext(): boolean;
}

interface Aggregator<T> {
  createIterator(): Iterator<T>;
}

// Concrete collection
class BookCollection implements Aggregator<string> {
  private books: string[] = [];

  addBook(book: string) {
    this.books.push(book);
  }

  createIterator(): Iterator<string> {
    return new BookIterator(this.books);
  }
}

// Concrete iterator
class BookIterator implements Iterator<string> {
  private index = 0;

  constructor(private books: string[]) {}

  next(): string {
    return this.books[this.index++];
  }

  hasNext(): boolean {
    return this.index < this.books.length;
  }
}

// Usage
const collection = new BookCollection();
collection.addBook('Design Patterns');
collection.addBook('Clean Code');
collection.addBook('The Pragmatic Programmer');

const iterator = collection.createIterator();
while (iterator.hasNext()) {
  console.log(iterator.next());
}
```

### Modern JavaScript Iterator

```typescript
class Range {
  constructor(
    private start: number,
    private end: number,
    private step: number = 1
  ) {}

  *[Symbol.iterator]() {
    for (let i = this.start; i <= this.end; i += this.step) {
      yield i;
    }
  }
}

// Usage
const range = new Range(1, 10, 2);
for (const num of range) {
  console.log(num); // 1, 3, 5, 7, 9
}

// Convert to array
console.log([...range]); // [1, 3, 5, 7, 9]
```

**When to use:**
- Traverse collections
- Multiple simultaneous traversals
- Uniform interface for different collections
- Hide implementation details

## Chain of Responsibility

### Purpose
Pass requests along a chain of handlers. Each handler decides either to process the request or to pass it to the next handler.

### Implementation

```typescript
// Handler interface
interface Handler {
  setNext(handler: Handler): Handler;
  handle(request: string): string | null;
}

// Abstract handler
abstract class AbstractHandler implements Handler {
  private nextHandler?: Handler;

  setNext(handler: Handler): Handler {
    this.nextHandler = handler;
    return handler;
  }

  handle(request: string): string | null {
    if (this.nextHandler) {
      return this.nextHandler.handle(request);
    }
    return null;
  }
}

// Concrete handlers
class AuthenticationHandler extends AbstractHandler {
  handle(request: string): string | null {
    if (request.includes('auth')) {
      return `AuthenticationHandler: Handling ${request}`;
    }
    return super.handle(request);
  }
}

class ValidationHandler extends AbstractHandler {
  handle(request: string): string | null {
    if (request.includes('validate')) {
      return `ValidationHandler: Handling ${request}`;
    }
    return super.handle(request);
  }
}

class LoggingHandler extends AbstractHandler {
  handle(request: string): string | null {
    if (request.includes('log')) {
      return `LoggingHandler: Handling ${request}`;
    }
    return super.handle(request);
  }
}

// Usage
const auth = new AuthenticationHandler();
const validation = new ValidationHandler();
const logging = new LoggingHandler();

auth.setNext(validation).setNext(logging);

console.log(auth.handle('auth:user'));      // AuthenticationHandler
console.log(auth.handle('validate:input')); // ValidationHandler
console.log(auth.handle('log:event'));      // LoggingHandler
console.log(auth.handle('unknown'));        // null
```

### Middleware Example

```typescript
type Middleware = (
  request: Request,
  next: () => Response
) => Response;

interface Request {
  url: string;
  headers: Record<string, string>;
  body?: any;
}

interface Response {
  status: number;
  body: any;
}

class MiddlewareChain {
  private middlewares: Middleware[] = [];

  use(middleware: Middleware) {
    this.middlewares.push(middleware);
    return this;
  }

  execute(request: Request): Response {
    let index = 0;

    const next = (): Response => {
      if (index >= this.middlewares.length) {
        return { status: 404, body: 'Not Found' };
      }

      const middleware = this.middlewares[index++];
      return middleware(request, next);
    };

    return next();
  }
}

// Middleware functions
const authMiddleware: Middleware = (req, next) => {
  console.log('Checking authentication...');
  if (!req.headers['authorization']) {
    return { status: 401, body: 'Unauthorized' };
  }
  return next();
};

const loggingMiddleware: Middleware = (req, next) => {
  console.log(`${new Date().toISOString()} - ${req.url}`);
  return next();
};

const rateLimitMiddleware: Middleware = (req, next) => {
  console.log('Checking rate limit...');
  // Check rate limit logic
  return next();
};

// Usage
const chain = new MiddlewareChain();
chain
  .use(loggingMiddleware)
  .use(rateLimitMiddleware)
  .use(authMiddleware);

const request: Request = {
  url: '/api/users',
  headers: { authorization: 'Bearer token' }
};

const response = chain.execute(request);
```

**When to use:**
- Middleware pipelines
- Request processing
- Event bubbling
- Validation chains

## Comparison

| Pattern | Purpose | Example |
|---------|---------|---------|
| **Observer** | Notify dependents | Event listeners |
| **Strategy** | Swap algorithms | Payment methods |
| **Command** | Encapsulate requests | Undo/redo |
| **State** | Change behavior with state | Vending machine |
| **Iterator** | Traverse collections | For...of loops |
| **Chain of Responsibility** | Pass along chain | Middleware |

## Best Practices

### ✅ Do

- Use observer for event-driven systems
- Use strategy for interchangeable algorithms
- Use command for undo/redo
- Use state for state-dependent behavior
- Use iterator for collection traversal
- Use chain of responsibility for request pipelines

### ❌ Don't

- Create too many observers (memory leaks)
- Over-complicate with unnecessary patterns
- Forget to unsubscribe observers
- Create circular dependencies in chains

## Further Reading

- [Creational Patterns](./creational.md)
- [Structural Patterns](./structural.md)
- [Frontend Patterns](./frontend-patterns.md)

---

*Previous: [Structural Patterns ←](./structural.md)*

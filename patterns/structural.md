# Structural Design Patterns

## Introduction

Structural patterns explain how to assemble objects and classes into larger structures while keeping these structures flexible and efficient. They help ensure that when one part changes, the entire structure doesn't need to change.

## Adapter Pattern

### Purpose
Convert one interface into another that clients expect. Allows incompatible interfaces to work together.

### Implementation

```typescript
// Target interface (what client expects)
interface MediaPlayer {
  play(fileName: string): void;
}

// Adaptee (existing incompatible interface)
class VLCPlayer {
  playVLC(fileName: string) {
    console.log(`Playing VLC file: ${fileName}`);
  }
}

class MP4Player {
  playMP4(fileName: string) {
    console.log(`Playing MP4 file: ${fileName}`);
  }
}

// Adapter
class MediaAdapter implements MediaPlayer {
  private vlcPlayer: VLCPlayer;
  private mp4Player: MP4Player;

  constructor(private audioType: string) {
    if (audioType === 'vlc') {
      this.vlcPlayer = new VLCPlayer();
    } else if (audioType === 'mp4') {
      this.mp4Player = new MP4Player();
    }
  }

  play(fileName: string) {
    if (this.audioType === 'vlc') {
      this.vlcPlayer.playVLC(fileName);
    } else if (this.audioType === 'mp4') {
      this.mp4Player.playMP4(fileName);
    }
  }
}

// Client
class AudioPlayer implements MediaPlayer {
  play(fileName: string) {
    const fileType = fileName.split('.').pop();

    if (fileType === 'mp3') {
      console.log(`Playing MP3 file: ${fileName}`);
    } else if (fileType === 'vlc' || fileType === 'mp4') {
      const adapter = new MediaAdapter(fileType);
      adapter.play(fileName);
    } else {
      console.log(`Invalid format: ${fileType}`);
    }
  }
}

// Usage
const player = new AudioPlayer();
player.play('song.mp3');  // Direct play
player.play('movie.mp4'); // Through adapter
player.play('video.vlc'); // Through adapter
```

### Real-World Example: API Adapter

```typescript
// Old API
class OldPaymentAPI {
  makePayment(amount: number, currency: string) {
    console.log(`Old API: Processing ${currency} ${amount}`);
    return { success: true, transactionId: '123' };
  }
}

// New API interface
interface PaymentProcessor {
  processPayment(payment: Payment): Promise<PaymentResult>;
}

interface Payment {
  amount: number;
  currency: string;
  method: string;
}

interface PaymentResult {
  success: boolean;
  transactionId: string;
}

// Adapter
class PaymentAdapter implements PaymentProcessor {
  constructor(private oldAPI: OldPaymentAPI) {}

  async processPayment(payment: Payment): Promise<PaymentResult> {
    // Adapt new interface to old API
    const result = this.oldAPI.makePayment(payment.amount, payment.currency);
    return {
      success: result.success,
      transactionId: result.transactionId
    };
  }
}

// Usage
const oldAPI = new OldPaymentAPI();
const adapter = new PaymentAdapter(oldAPI);

await adapter.processPayment({
  amount: 100,
  currency: 'USD',
  method: 'credit-card'
});
```

**When to use:**
- Integrate legacy code
- Work with third-party libraries
- Incompatible interfaces
- Reuse existing classes

## Decorator Pattern

### Purpose
Add new functionality to objects dynamically without altering their structure.

### Implementation

```typescript
// Component interface
interface Coffee {
  cost(): number;
  description(): string;
}

// Concrete component
class SimpleCoffee implements Coffee {
  cost() {
    return 5;
  }

  description() {
    return 'Simple coffee';
  }
}

// Decorator base
abstract class CoffeeDecorator implements Coffee {
  constructor(protected coffee: Coffee) {}

  abstract cost(): number;
  abstract description(): string;
}

// Concrete decorators
class MilkDecorator extends CoffeeDecorator {
  cost() {
    return this.coffee.cost() + 2;
  }

  description() {
    return this.coffee.description() + ', milk';
  }
}

class SugarDecorator extends CoffeeDecorator {
  cost() {
    return this.coffee.cost() + 1;
  }

  description() {
    return this.coffee.description() + ', sugar';
  }
}

class WhipDecorator extends CoffeeDecorator {
  cost() {
    return this.coffee.cost() + 3;
  }

  description() {
    return this.coffee.description() + ', whip';
  }
}

// Usage
let coffee: Coffee = new SimpleCoffee();
console.log(`${coffee.description()}: $${coffee.cost()}`);
// Simple coffee: $5

coffee = new MilkDecorator(coffee);
console.log(`${coffee.description()}: $${coffee.cost()}`);
// Simple coffee, milk: $7

coffee = new SugarDecorator(coffee);
coffee = new WhipDecorator(coffee);
console.log(`${coffee.description()}: $${coffee.cost()}`);
// Simple coffee, milk, sugar, whip: $11
```

### Modern TypeScript Decorators

```typescript
// Method decorator
function log(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;

  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey} with args:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Result:`, result);
    return result;
  };

  return descriptor;
}

// Usage
class Calculator {
  @log
  add(a: number, b: number) {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(5, 3);
// Logs: Calling add with args: [5, 3]
// Logs: Result: 8
```

**When to use:**
- Add functionality without modifying code
- Multiple optional features
- Runtime composition
- Alternative to subclassing

## Facade Pattern

### Purpose
Provide a simplified interface to a complex subsystem.

### Implementation

```typescript
// Complex subsystems
class CPU {
  freeze() {
    console.log('CPU: Freezing...');
  }

  jump(position: number) {
    console.log(`CPU: Jumping to ${position}`);
  }

  execute() {
    console.log('CPU: Executing...');
  }
}

class Memory {
  load(position: number, data: string) {
    console.log(`Memory: Loading ${data} at ${position}`);
  }
}

class HardDrive {
  read(sector: number, size: number): string {
    console.log(`HardDrive: Reading sector ${sector}, size ${size}`);
    return 'boot data';
  }
}

// Facade
class ComputerFacade {
  private cpu: CPU;
  private memory: Memory;
  private hardDrive: HardDrive;

  constructor() {
    this.cpu = new CPU();
    this.memory = new Memory();
    this.hardDrive = new HardDrive();
  }

  start() {
    console.log('Starting computer...');
    this.cpu.freeze();
    this.memory.load(0, this.hardDrive.read(0, 1024));
    this.cpu.jump(0);
    this.cpu.execute();
    console.log('Computer started!');
  }
}

// Usage
const computer = new ComputerFacade();
computer.start(); // Simple interface hides complexity
```

### Frontend Example

```typescript
// Complex API subsystems
class AuthAPI {
  async login(credentials: Credentials) {
    // Complex auth logic
  }

  async refresh Token() {
    // Refresh logic
  }
}

class UserAPI {
  async getProfile(userId: string) {
    // Get user data
  }

  async updateProfile(data: ProfileData) {
    // Update logic
  }
}

class PreferencesAPI {
  async getPreferences(userId: string) {
    // Get preferences
  }
}

// Facade
class UserService {
  private authAPI = new AuthAPI();
  private userAPI = new UserAPI();
  private preferencesAPI = new PreferencesAPI();

  async loginAndFetchUserData(credentials: Credentials) {
    // Simplified interface for common operation
    const authResult = await this.authAPI.login(credentials);
    const profile = await this.userAPI.getProfile(authResult.userId);
    const preferences = await this.preferencesAPI.getPreferences(authResult.userId);

    return {
      auth: authResult,
      profile,
      preferences
    };
  }
}

// Usage
const userService = new UserService();
const userData = await userService.loginAndFetchUserData(credentials);
```

**When to use:**
- Simplify complex systems
- Layer your application
- Reduce dependencies
- Provide cleaner API

## Composite Pattern

### Purpose
Compose objects into tree structures to represent part-whole hierarchies.

### Implementation

```typescript
// Component interface
interface FileSystemItem {
  getName(): string;
  getSize(): number;
  print(indent: string): void;
}

// Leaf
class File implements FileSystemItem {
  constructor(
    private name: string,
    private size: number
  ) {}

  getName(): string {
    return this.name;
  }

  getSize(): number {
    return this.size;
  }

  print(indent: string = '') {
    console.log(`${indent}📄 ${this.name} (${this.size}KB)`);
  }
}

// Composite
class Directory implements FileSystemItem {
  private children: FileSystemItem[] = [];

  constructor(private name: string) {}

  add(item: FileSystemItem) {
    this.children.push(item);
  }

  remove(item: FileSystemItem) {
    const index = this.children.indexOf(item);
    if (index > -1) {
      this.children.splice(index, 1);
    }
  }

  getName(): string {
    return this.name;
  }

  getSize(): number {
    return this.children.reduce((sum, child) => sum + child.getSize(), 0);
  }

  print(indent: string = '') {
    console.log(`${indent}📁 ${this.name} (${this.getSize()}KB)`);
    this.children.forEach(child => child.print(indent + '  '));
  }
}

// Usage
const root = new Directory('root');
const home = new Directory('home');
const documents = new Directory('documents');

home.add(new File('resume.pdf', 100));
home.add(new File('photo.jpg', 500));

documents.add(new File('report.docx', 200));
documents.add(new File('presentation.pptx', 300));

home.add(documents);
root.add(home);
root.add(new File('boot.sys', 50));

root.print();
// 📁 root (1150KB)
//   📁 home (1100KB)
//     📄 resume.pdf (100KB)
//     📄 photo.jpg (500KB)
//     📁 documents (500KB)
//       📄 report.docx (200KB)
//       📄 presentation.pptx (300KB)
//   📄 boot.sys (50KB)
```

### React Component Example

```typescript
// Component interface
interface UIComponent {
  render(): ReactNode;
}

// Leaf components
class Button implements UIComponent {
  constructor(private text: string) {}

  render() {
    return <button>{this.text}</button>;
  }
}

class Input implements UIComponent {
  constructor(private placeholder: string) {}

  render() {
    return <input placeholder={this.placeholder} />;
  }
}

// Composite
class Panel implements UIComponent {
  private children: UIComponent[] = [];

  constructor(private title: string) {}

  add(component: UIComponent) {
    this.children.push(component);
  }

  render() {
    return (
      <div className="panel">
        <h3>{this.title}</h3>
        {this.children.map(child => child.render())}
      </div>
    );
  }
}

// Usage
const loginPanel = new Panel('Login');
loginPanel.add(new Input('Username'));
loginPanel.add(new Input('Password'));
loginPanel.add(new Button('Login'));
```

**When to use:**
- Tree structures
- Part-whole hierarchies
- Uniform treatment of objects
- UI components

## Proxy Pattern

### Purpose
Provide a surrogate or placeholder for another object to control access to it.

### Implementation

```typescript
// Subject interface
interface Image {
  display(): void;
}

// Real subject
class RealImage implements Image {
  constructor(private fileName: string) {
    this.loadFromDisk();
  }

  private loadFromDisk() {
    console.log(`Loading image: ${this.fileName}`);
  }

  display() {
    console.log(`Displaying image: ${this.fileName}`);
  }
}

// Proxy
class ProxyImage implements Image {
  private realImage: RealImage | null = null;

  constructor(private fileName: string) {}

  display() {
    // Lazy loading - only create real image when needed
    if (!this.realImage) {
      this.realImage = new RealImage(this.fileName);
    }
    this.realImage.display();
  }
}

// Usage
const image1 = new ProxyImage('photo1.jpg');
const image2 = new ProxyImage('photo2.jpg');

// Images not loaded yet
console.log('Images created');

// Load and display only when needed
image1.display(); // Loading image: photo1.jpg, Displaying image: photo1.jpg
image1.display(); // Displaying image: photo1.jpg (already loaded)
```

### Different Proxy Types

```typescript
// 1. Virtual Proxy (Lazy Loading)
class VirtualProxy {
  private realObject: ExpensiveObject | null = null;

  execute() {
    if (!this.realObject) {
      this.realObject = new ExpensiveObject();
    }
    return this.realObject.execute();
  }
}

// 2. Protection Proxy (Access Control)
class ProtectionProxy {
  constructor(
    private realObject: SensitiveObject,
    private user: User
  ) {}

  execute() {
    if (this.user.hasPermission('admin')) {
      return this.realObject.execute();
    }
    throw new Error('Access denied');
  }
}

// 3. Caching Proxy
class CachingProxy {
  private cache = new Map<string, any>();

  constructor(private api: API) {}

  async fetch(url: string) {
    if (this.cache.has(url)) {
      console.log('Cache hit');
      return this.cache.get(url);
    }

    console.log('Cache miss');
    const result = await this.api.fetch(url);
    this.cache.set(url, result);
    return result;
  }
}

// 4. Logging Proxy
class LoggingProxy {
  constructor(private realObject: any) {}

  execute(...args: any[]) {
    console.log(`Method called with:`, args);
    const result = this.realObject.execute(...args);
    console.log(`Method returned:`, result);
    return result;
  }
}
```

**When to use:**
- Lazy initialization
- Access control
- Caching
- Logging
- Remote objects

## Bridge Pattern

### Purpose
Separate abstraction from implementation so both can vary independently.

### Implementation

```typescript
// Implementation interface
interface Device {
  isEnabled(): boolean;
  enable(): void;
  disable(): void;
  getVolume(): number;
  setVolume(percent: number): void;
}

// Concrete implementations
class TV implements Device {
  private on = false;
  private volume = 30;

  isEnabled() {
    return this.on;
  }

  enable() {
    this.on = true;
    console.log('TV is now ON');
  }

  disable() {
    this.on = false;
    console.log('TV is now OFF');
  }

  getVolume() {
    return this.volume;
  }

  setVolume(percent: number) {
    this.volume = percent;
    console.log(`TV volume: ${percent}%`);
  }
}

class Radio implements Device {
  private on = false;
  private volume = 50;

  isEnabled() {
    return this.on;
  }

  enable() {
    this.on = true;
    console.log('Radio is now ON');
  }

  disable() {
    this.on = false;
    console.log('Radio is now OFF');
  }

  getVolume() {
    return this.volume;
  }

  setVolume(percent: number) {
    this.volume = percent;
    console.log(`Radio volume: ${percent}%`);
  }
}

// Abstraction
class RemoteControl {
  constructor(protected device: Device) {}

  togglePower() {
    if (this.device.isEnabled()) {
      this.device.disable();
    } else {
      this.device.enable();
    }
  }

  volumeUp() {
    this.device.setVolume(this.device.getVolume() + 10);
  }

  volumeDown() {
    this.device.setVolume(this.device.getVolume() - 10);
  }
}

// Refined abstraction
class AdvancedRemoteControl extends RemoteControl {
  mute() {
    console.log('Muting device');
    this.device.setVolume(0);
  }
}

// Usage
const tv = new TV();
const remote = new RemoteControl(tv);
remote.togglePower();  // TV is now ON
remote.volumeUp();     // TV volume: 40%

const radio = new Radio();
const advancedRemote = new AdvancedRemoteControl(radio);
advancedRemote.togglePower();  // Radio is now ON
advancedRemote.mute();         // Muting device, Radio volume: 0%
```

**When to use:**
- Avoid permanent binding between abstraction and implementation
- Both abstraction and implementation should be extensible
- Changes in implementation shouldn't affect clients
- Multiple implementations to choose from

## Comparison

| Pattern | Purpose | Example |
|---------|---------|---------|
| **Adapter** | Convert interface | Old API → New API |
| **Decorator** | Add functionality | Coffee + Milk |
| **Facade** | Simplify interface | Complex system → Simple API |
| **Composite** | Tree structure | File system, UI components |
| **Proxy** | Control access | Lazy loading, Caching |
| **Bridge** | Separate abstraction/implementation | Remote → Device |

## Best Practices

### ✅ Do

- Use adapter for incompatible interfaces
- Use decorator for flexible features
- Use facade to simplify complex systems
- Use composite for tree structures
- Use proxy for lazy loading/access control
- Use bridge to decouple abstraction from implementation

### ❌ Don't

- Over-complicate simple scenarios
- Use structural patterns when not needed
- Create too many layers of abstraction
- Forget about performance implications

## Further Reading

- [Creational Patterns](./creational.md)
- [Behavioral Patterns](./behavioral.md)
- [Frontend Patterns](./frontend-patterns.md)

---

*Next: [Behavioral Patterns →](./behavioral.md)*

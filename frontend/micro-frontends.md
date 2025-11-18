# Micro-Frontends Architecture

## Introduction

Micro-frontends extend the concept of microservices to frontend development. The idea is to decompose a frontend application into smaller, semi-independent "micro applications" that can be developed, tested, and deployed independently.

## Core Concepts

### What are Micro-Frontends?

An architectural style where independently deliverable frontend applications are composed into a greater whole.

**Key characteristics:**
- **Technology agnostic** - Teams can choose their own tech stack
- **Independent deployment** - Deploy without coordinating with other teams
- **Team autonomy** - Teams own features end-to-end
- **Isolated code** - No shared state or dependencies by default

### Why Micro-Frontends?

**Benefits:**
- Scale development across multiple teams
- Independent deployment and releases
- Technology flexibility
- Easier to understand and maintain
- Incremental upgrades

**Challenges:**
- Increased complexity
- Performance overhead
- Consistency across apps
- Shared dependencies management
- Cross-cutting concerns

## Implementation Approaches

### 1. Build-Time Integration

Compile micro-frontends together at build time.

```json
// package.json
{
  "dependencies": {
    "@company/header": "^1.0.0",
    "@company/products": "^2.0.0",
    "@company/checkout": "^1.5.0"
  }
}
```

```tsx
// App.tsx
import { Header } from '@company/header';
import { Products } from '@company/products';
import { Checkout } from '@company/checkout';

const App = () => (
  <div>
    <Header />
    <Routes>
      <Route path="/products" element={<Products />} />
      <Route path="/checkout" element={<Checkout />} />
    </Routes>
  </div>
);
```

**Pros:**
- Simple to implement
- Single deployment artifact
- Shared dependencies optimization

**Cons:**
- No independent deployment
- Must rebuild entire app for updates
- Version conflicts

**When to use:**
- Small teams
- Shared release cycle
- Simple integration needs

### 2. Run-Time Integration via iFrames

Each micro-frontend runs in an iframe.

```tsx
// Container app
const App = () => (
  <div>
    <iframe src="https://header.example.com" />
    <iframe src="https://products.example.com" />
    <iframe src="https://cart.example.com" />
  </div>
);
```

**Pros:**
- Complete isolation
- True independent deployment
- No shared dependencies
- Different tech stacks per iframe

**Cons:**
- Poor performance
- Difficult to make responsive
- Routing challenges
- Communication complexity

**When to use:**
- Maximum isolation required
- Legacy system integration
- Third-party content

### 3. Run-Time Integration via JavaScript

Load micro-frontends dynamically at runtime.

```tsx
// Container app
const MicroFrontend = ({ name, host }: { name: string; host: string }) => {
  const [loaded, setLoaded] = useState(false);

  useEffect(() => {
    const scriptId = `micro-frontend-script-${name}`;

    if (document.getElementById(scriptId)) {
      setLoaded(true);
      return;
    }

    const script = document.createElement('script');
    script.id = scriptId;
    script.src = `${host}/main.js`;
    script.onload = () => setLoaded(true);
    document.head.appendChild(script);

    return () => {
      document.getElementById(scriptId)?.remove();
    };
  }, [name, host]);

  return (
    <div id={`${name}-container`}>
      {!loaded && <Loading />}
    </div>
  );
};

// Usage
<MicroFrontend name="products" host="https://products.example.com" />
```

**Pros:**
- True independent deployment
- Runtime flexibility
- Technology agnostic

**Cons:**
- Complex implementation
- Duplicate dependencies
- Performance overhead
- Initial load time

### 4. Module Federation (Webpack 5)

Share code dynamically between applications.

```javascript
// products/webpack.config.js
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'products',
      filename: 'remoteEntry.js',
      exposes: {
        './ProductList': './src/components/ProductList',
        './ProductDetail': './src/components/ProductDetail',
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true },
      },
    }),
  ],
};

// container/webpack.config.js
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'container',
      remotes: {
        products: 'products@https://products.example.com/remoteEntry.js',
        checkout: 'checkout@https://checkout.example.com/remoteEntry.js',
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true },
      },
    }),
  ],
};

// Container app usage
import React, { lazy, Suspense } from 'react';

const ProductList = lazy(() => import('products/ProductList'));
const Checkout = lazy(() => import('checkout/CheckoutForm'));

const App = () => (
  <Suspense fallback={<Loading />}>
    <Routes>
      <Route path="/products" element={<ProductList />} />
      <Route path="/checkout" element={<Checkout />} />
    </Routes>
  </Suspense>
);
```

**Pros:**
- Dynamic module loading
- Shared dependencies optimization
- True micro-frontend architecture
- Independent deployment

**Cons:**
- Webpack-specific
- Complex configuration
- Learning curve
- Version management

**When to use:**
- Modern applications
- Webpack-based projects
- Need shared dependencies
- Multiple independent teams

### 5. Web Components

Standards-based approach using Custom Elements.

```tsx
// products/ProductCard.ts
class ProductCard extends HTMLElement {
  connectedCallback() {
    this.render();
  }

  render() {
    const productId = this.getAttribute('product-id');
    this.innerHTML = `
      <div class="product-card">
        <h3>Product ${productId}</h3>
        <button>Add to Cart</button>
      </div>
    `;
  }
}

customElements.define('product-card', ProductCard);

// Container app - any framework or vanilla JS
<product-card product-id="123"></product-card>
```

**Pros:**
- Framework agnostic
- Web standards
- True encapsulation
- Browser support

**Cons:**
- Limited framework integration
- SEO challenges
- Styling isolation complexity

**When to use:**
- Mixed technology stack
- Long-term stability
- Framework independence

## Communication Patterns

### 1. Custom Events

```typescript
// Micro-frontend A - emits event
const addToCartButton = document.querySelector('#add-to-cart');
addToCartButton?.addEventListener('click', () => {
  window.dispatchEvent(
    new CustomEvent('cart:item-added', {
      detail: { productId: '123', quantity: 1 },
    })
  );
});

// Micro-frontend B - listens for event
window.addEventListener('cart:item-added', (event: Event) => {
  const { productId, quantity } = (event as CustomEvent).detail;
  updateCartCount(quantity);
});
```

### 2. Shared State (Pub/Sub)

```typescript
// Shared event bus
class EventBus {
  private listeners = new Map<string, Set<Function>>();

  subscribe(event: string, callback: Function) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(callback);

    return () => this.unsubscribe(event, callback);
  }

  unsubscribe(event: string, callback: Function) {
    this.listeners.get(event)?.delete(callback);
  }

  publish(event: string, data?: any) {
    this.listeners.get(event)?.forEach(callback => callback(data));
  }
}

// Make available globally
window.eventBus = new EventBus();

// Micro-frontend A
window.eventBus.publish('user:login', { userId: '123' });

// Micro-frontend B
window.eventBus.subscribe('user:login', (user) => {
  console.log('User logged in:', user);
});
```

### 3. Props/Callbacks

```tsx
// Container passes props to micro-frontends
<MicroFrontend
  name="products"
  user={currentUser}
  onAddToCart={handleAddToCart}
/>
```

### 4. Shared Storage

```typescript
// LocalStorage/SessionStorage
localStorage.setItem('cart', JSON.stringify(cartItems));

// IndexedDB for complex data
const db = await openDB('app-db', 1);
await db.put('cart', cartItems);

// Cookies for cross-domain
document.cookie = 'user=123; domain=.example.com';
```

## Routing Strategies

### 1. Container-Level Routing

Container app handles all routing.

```tsx
// Container app
const App = () => (
  <Router>
    <Header />
    <Routes>
      <Route path="/products/*" element={<ProductsMicroFrontend />} />
      <Route path="/checkout/*" element={<CheckoutMicroFrontend />} />
      <Route path="/account/*" element={<AccountMicroFrontend />} />
    </Routes>
  </Router>
);
```

### 2. Micro-Frontend-Level Routing

Each micro-frontend handles its own routes.

```tsx
// Products micro-frontend
const ProductsApp = ({ basePath }: { basePath: string }) => (
  <Router basename={basePath}>
    <Routes>
      <Route path="/" element={<ProductList />} />
      <Route path="/:id" element={<ProductDetail />} />
      <Route path="/categories" element={<Categories />} />
    </Routes>
  </Router>
);

// Container
<ProductsApp basePath="/products" />
```

## Shared Dependencies

### 1. Vendor Splitting

```javascript
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/](react|react-dom|react-router-dom)[\\/]/,
          name: 'vendor',
          chunks: 'all',
        },
      },
    },
  },
};
```

### 2. External Dependencies

```javascript
// webpack.config.js
module.exports = {
  externals: {
    react: 'React',
    'react-dom': 'ReactDOM',
  },
};

// Load from CDN in container
<script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
```

### 3. Module Federation Shared

```javascript
new ModuleFederationPlugin({
  shared: {
    react: {
      singleton: true,
      requiredVersion: '^18.0.0',
    },
    'react-dom': {
      singleton: true,
      requiredVersion: '^18.0.0',
    },
  },
});
```

## Styling Strategies

### 1. CSS Modules

```tsx
// products.module.css
.container {
  padding: 20px;
}

// Component
import styles from './products.module.css';

const Products = () => <div className={styles.container}>...</div>;
```

### 2. CSS-in-JS with Scoping

```tsx
import styled from 'styled-components';

const Container = styled.div`
  padding: 20px;
  /* Automatically scoped */
`;
```

### 3. Shadow DOM

```typescript
class ProductCard extends HTMLElement {
  connectedCallback() {
    const shadow = this.attachShadow({ mode: 'open' });
    shadow.innerHTML = `
      <style>
        :host {
          display: block;
          padding: 20px;
        }
      </style>
      <div class="card">Content</div>
    `;
  }
}
```

### 4. Prefix Convention

```css
/* products micro-frontend */
.products-card { }
.products-button { }

/* checkout micro-frontend */
.checkout-form { }
.checkout-button { }
```

## Design System Integration

```typescript
// Shared design system package
// @company/design-system

export { Button, Input, Card } from './components';
export { theme } from './theme';
export { GlobalStyles } from './styles';

// Each micro-frontend uses it
import { Button, theme } from '@company/design-system';
```

## Best Practices

### ✅ Do

- Define clear boundaries between micro-frontends
- Establish communication protocols
- Share design system and common components
- Implement monitoring and error tracking
- Version your micro-frontends
- Document integration points
- Use feature flags for gradual rollout
- Implement proper error boundaries

### ❌ Don't

- Share state directly between micro-frontends
- Create tight coupling
- Duplicate common utilities excessively
- Ignore performance implications
- Mix multiple integration approaches
- Skip versioning strategy
- Forget about accessibility
- Ignore SEO requirements

## Deployment Strategies

### Independent Deployment

```yaml
# CI/CD for products micro-frontend
name: Deploy Products
on:
  push:
    branches: [main]
    paths:
      - 'packages/products/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: npm install
      - run: npm run build
      - run: aws s3 sync dist/ s3://products-mfe/
```

### Versioning

```html
<!-- Container app -->
<script src="https://products.example.com/v1/remoteEntry.js"></script>
<script src="https://checkout.example.com/v2/remoteEntry.js"></script>
```

## Monitoring & Debugging

```typescript
// Error tracking for micro-frontends
window.addEventListener('error', (event) => {
  const microFrontend = event.target?.closest('[data-mfe]')?.getAttribute('data-mfe');

  analytics.track('mfe-error', {
    microFrontend,
    error: event.error,
    message: event.message,
  });
});

// Performance monitoring
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`${entry.name}: ${entry.duration}ms`);
  }
});

observer.observe({ entryTypes: ['measure'] });
```

## Further Reading

- [Application Structure](./app-structure.md)
- [Performance Architecture](./performance.md)
- [Design Systems](./design-systems.md)

---

*Next: [Design Systems →](./design-systems.md)*

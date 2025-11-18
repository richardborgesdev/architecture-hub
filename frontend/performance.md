# Performance Architecture

## Introduction

Performance should be a core architectural concern, not an afterthought. This guide covers strategies for building fast, efficient frontend applications.

## Core Web Vitals

### 1. Largest Contentful Paint (LCP)

**Target: < 2.5 seconds**

Measures loading performance. The largest element in viewport should render quickly.

**Optimization strategies:**
- Optimize images (WebP, lazy loading)
- Reduce server response time
- Eliminate render-blocking resources
- Implement CDN for static assets
- Use resource hints (preload, prefetch)

```tsx
// Preload critical resources
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin />
<link rel="preload" href="/hero-image.webp" as="image" />

// Lazy load images
<img
  src="placeholder.jpg"
  data-src="actual-image.jpg"
  loading="lazy"
  alt="Description"
/>
```

### 2. First Input Delay (FID)

**Target: < 100 milliseconds**

Measures interactivity. Time from user interaction to browser response.

**Optimization strategies:**
- Break up long tasks
- Use web workers for heavy computation
- Minimize JavaScript execution time
- Code splitting
- Defer non-critical JavaScript

```tsx
// Use web workers for heavy computation
const worker = new Worker('/workers/data-processor.js');

worker.postMessage({ data: largeDataset });

worker.onmessage = (e) => {
  setProcessedData(e.data);
};

// Break up long tasks
async function processLargeList(items) {
  for (let i = 0; i < items.length; i++) {
    await processItem(items[i]);

    // Yield to main thread every 50 items
    if (i % 50 === 0) {
      await new Promise(resolve => setTimeout(resolve, 0));
    }
  }
}
```

### 3. Cumulative Layout Shift (CLS)

**Target: < 0.1**

Measures visual stability. Avoid unexpected layout shifts.

**Optimization strategies:**
- Set dimensions for images and videos
- Reserve space for dynamic content
- Avoid inserting content above existing content
- Use CSS transforms instead of layout properties

```tsx
// Reserve space for images
<img
  src="image.jpg"
  width="800"
  height="600"
  alt="Description"
  style={{ aspectRatio: '800/600' }}
/>

// Reserve space for dynamic content
<div style={{ minHeight: '200px' }}>
  {loading ? <Skeleton /> : <Content />}
</div>

// Use transform instead of top/left for animations
.animated {
  transform: translateX(100px); /* Better performance */
  /* top: 100px; */ /* Triggers layout */
}
```

## Bundle Optimization

### 1. Code Splitting

```tsx
// Route-based splitting
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Analytics = lazy(() => import('./pages/Analytics'));

// Component-based splitting
const HeavyChart = lazy(() => import('./components/HeavyChart'));

// Dynamic imports
const loadModule = async () => {
  const module = await import('./utils/heavy-library');
  module.doSomething();
};

// Prefetch on hover
<Link
  to="/dashboard"
  onMouseEnter={() => import('./pages/Dashboard')}
>
  Dashboard
</Link>
```

### 2. Tree Shaking

```tsx
// ❌ Bad: Imports entire library
import _ from 'lodash';
const result = _.debounce(fn, 100);

// ✅ Good: Import only what you need
import debounce from 'lodash/debounce';
const result = debounce(fn, 100);

// Even better: Use modern alternatives
import { debounce } from '@/utils/debounce'; // Custom implementation
```

### 3. Bundle Analysis

```bash
# Analyze bundle size
npm run build -- --analyze

# Or use webpack-bundle-analyzer
npx webpack-bundle-analyzer dist/stats.json
```

```javascript
// vite.config.ts
import { defineConfig } from 'vite';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom'],
          'ui-library': ['@mui/material'],
          'charts': ['recharts', 'd3'],
        },
      },
    },
  },
  plugins: [
    visualizer({ open: true }),
  ],
});
```

## Rendering Strategies

### 1. Client-Side Rendering (CSR)

```tsx
// Traditional SPA
const App = () => {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetchData().then(setData);
  }, []);

  if (!data) return <Loading />;
  return <Dashboard data={data} />;
};
```

**Pros:**
- Rich interactivity
- Simple deployment
- Reduced server load

**Cons:**
- Poor initial load time
- SEO challenges
- Requires JavaScript

### 2. Server-Side Rendering (SSR)

```tsx
// Next.js example
export async function getServerSideProps(context) {
  const data = await fetchData();

  return {
    props: { data },
  };
}

const Page = ({ data }) => {
  return <Dashboard data={data} />;
};
```

**Pros:**
- Fast initial load
- Better SEO
- Works without JavaScript

**Cons:**
- Increased server load
- Complex caching
- TTFB can be slow

### 3. Static Site Generation (SSG)

```tsx
// Next.js example
export async function getStaticProps() {
  const data = await fetchData();

  return {
    props: { data },
    revalidate: 60, // ISR: Revalidate every 60s
  };
}

const Page = ({ data }) => {
  return <Content data={data} />;
};
```

**Pros:**
- Fastest load time
- Excellent SEO
- Low server cost

**Cons:**
- Build time increases
- Not suitable for dynamic content
- Requires rebuild for updates

### 4. Incremental Static Regeneration (ISR)

```tsx
// Combine SSG benefits with dynamic updates
export async function getStaticProps() {
  const data = await fetchData();

  return {
    props: { data },
    revalidate: 10, // Regenerate page every 10 seconds
  };
}
```

### 5. Streaming SSR

```tsx
// React 18 Suspense streaming
import { renderToReadableStream } from 'react-dom/server';

const stream = await renderToReadableStream(
  <App />,
  {
    bootstrapScripts: ['/main.js'],
  }
);
```

## Data Fetching Optimization

### 1. Parallel Requests

```tsx
// ❌ Bad: Sequential requests
const Component = () => {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState(null);

  useEffect(() => {
    fetchUser().then(user => {
      setUser(user);
      fetchPosts(user.id).then(setPosts);
    });
  }, []);
};

// ✅ Good: Parallel requests
const Component = () => {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState(null);

  useEffect(() => {
    Promise.all([
      fetchUser(),
      fetchPosts(),
    ]).then(([user, posts]) => {
      setUser(user);
      setPosts(posts);
    });
  }, []);
};

// Even better: Use React Query
const Component = () => {
  const { data: user } = useQuery(['user'], fetchUser);
  const { data: posts } = useQuery(['posts'], fetchPosts);
};
```

### 2. Request Deduplication

```tsx
// React Query automatically deduplicates
const ComponentA = () => {
  const { data } = useQuery(['user', '123'], () => fetchUser('123'));
};

const ComponentB = () => {
  const { data } = useQuery(['user', '123'], () => fetchUser('123'));
  // Same query, only one request made
};
```

### 3. Prefetching

```tsx
// Prefetch on hover
const UserLink = ({ userId }) => {
  const queryClient = useQueryClient();

  return (
    <Link
      to={`/users/${userId}`}
      onMouseEnter={() => {
        queryClient.prefetchQuery(['user', userId], () => fetchUser(userId));
      }}
    >
      View Profile
    </Link>
  );
};

// Prefetch on route
const routes = [
  {
    path: '/dashboard',
    element: <Dashboard />,
    loader: async () => {
      const data = await queryClient.fetchQuery(['dashboard'], fetchDashboard);
      return data;
    },
  },
];
```

### 4. Caching Strategy

```tsx
// Configure stale-while-revalidate
const { data } = useQuery(['products'], fetchProducts, {
  staleTime: 5 * 60 * 1000, // 5 minutes
  cacheTime: 10 * 60 * 1000, // 10 minutes
  refetchOnWindowFocus: false,
  refetchOnReconnect: true,
});

// Implement cache invalidation
const useUpdateProduct = () => {
  const queryClient = useQueryClient();

  return useMutation(updateProduct, {
    onSuccess: (data, variables) => {
      // Invalidate related queries
      queryClient.invalidateQueries(['products']);
      queryClient.invalidateQueries(['product', variables.id]);
    },
  });
};
```

## Image Optimization

### 1. Modern Formats

```tsx
// Use WebP with fallback
<picture>
  <source srcSet="image.webp" type="image/webp" />
  <source srcSet="image.jpg" type="image/jpeg" />
  <img src="image.jpg" alt="Description" />
</picture>

// Or use Next.js Image component
import Image from 'next/image';

<Image
  src="/image.jpg"
  alt="Description"
  width={800}
  height={600}
  quality={75}
  loading="lazy"
/>
```

### 2. Responsive Images

```tsx
<img
  src="image-400.jpg"
  srcSet="
    image-400.jpg 400w,
    image-800.jpg 800w,
    image-1200.jpg 1200w
  "
  sizes="(max-width: 600px) 400px, (max-width: 1200px) 800px, 1200px"
  alt="Description"
/>
```

### 3. Lazy Loading

```tsx
// Native lazy loading
<img src="image.jpg" loading="lazy" alt="Description" />

// Intersection Observer for custom lazy loading
const LazyImage = ({ src, alt }) => {
  const [imageSrc, setImageSrc] = useState('placeholder.jpg');
  const imgRef = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setImageSrc(src);
          observer.disconnect();
        }
      },
      { rootMargin: '50px' }
    );

    if (imgRef.current) {
      observer.observe(imgRef.current);
    }

    return () => observer.disconnect();
  }, [src]);

  return <img ref={imgRef} src={imageSrc} alt={alt} />;
};
```

### 4. Progressive Loading

```tsx
// Blur-up technique
const ProgressiveImage = ({ src, placeholder }) => {
  const [currentSrc, setCurrentSrc] = useState(placeholder);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const img = new Image();
    img.src = src;
    img.onload = () => {
      setCurrentSrc(src);
      setLoading(false);
    };
  }, [src]);

  return (
    <img
      src={currentSrc}
      alt=""
      style={{
        filter: loading ? 'blur(10px)' : 'none',
        transition: 'filter 0.3s',
      }}
    />
  );
};
```

## Memory Management

### 1. Cleanup Side Effects

```tsx
// ❌ Bad: Memory leak
useEffect(() => {
  const interval = setInterval(() => {
    updateData();
  }, 1000);
}, []);

// ✅ Good: Cleanup
useEffect(() => {
  const interval = setInterval(() => {
    updateData();
  }, 1000);

  return () => clearInterval(interval);
}, []);

// Cleanup subscriptions
useEffect(() => {
  const subscription = dataStream.subscribe(handleData);

  return () => subscription.unsubscribe();
}, []);
```

### 2. Avoid Memory Leaks

```tsx
// Component unmount flag
const Component = () => {
  const [data, setData] = useState(null);
  const isMounted = useRef(true);

  useEffect(() => {
    fetchData().then(result => {
      if (isMounted.current) {
        setData(result);
      }
    });

    return () => {
      isMounted.current = false;
    };
  }, []);
};

// Or use AbortController
useEffect(() => {
  const controller = new AbortController();

  fetch('/api/data', { signal: controller.signal })
    .then(res => res.json())
    .then(setData)
    .catch(err => {
      if (err.name !== 'AbortError') {
        handleError(err);
      }
    });

  return () => controller.abort();
}, []);
```

## Rendering Optimization

### 1. Virtualization

```tsx
import { FixedSizeList } from 'react-window';

const VirtualList = ({ items }) => (
  <FixedSizeList
    height={600}
    itemCount={items.length}
    itemSize={50}
    width="100%"
  >
    {({ index, style }) => (
      <div style={style}>
        {items[index].name}
      </div>
    )}
  </FixedSizeList>
);
```

### 2. Memoization

```tsx
// Prevent unnecessary re-renders
const ExpensiveComponent = memo(({ data, onAction }) => {
  return <div>{/* Complex rendering */}</div>;
}, (prevProps, nextProps) => {
  // Custom comparison
  return prevProps.data.id === nextProps.data.id;
});

// Memoize expensive calculations
const ProcessedData = ({ items }) => {
  const filtered = useMemo(
    () => items.filter(item => item.active).sort(),
    [items]
  );

  return <List items={filtered} />;
};
```

### 3. Debouncing & Throttling

```tsx
import { useDebouncedCallback } from 'use-debounce';

const SearchInput = () => {
  const [query, setQuery] = useState('');

  const debouncedSearch = useDebouncedCallback(
    (value) => {
      performSearch(value);
    },
    500
  );

  return (
    <input
      value={query}
      onChange={(e) => {
        setQuery(e.target.value);
        debouncedSearch(e.target.value);
      }}
    />
  );
};

// Throttle scroll events
const useThrottledScroll = (callback, delay) => {
  const lastRun = useRef(Date.now());

  useEffect(() => {
    const handleScroll = () => {
      const now = Date.now();
      if (now - lastRun.current >= delay) {
        callback();
        lastRun.current = now;
      }
    };

    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, [callback, delay]);
};
```

## Performance Monitoring

### 1. Web Vitals Tracking

```tsx
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

function sendToAnalytics(metric) {
  const body = JSON.stringify(metric);
  // Use sendBeacon if available
  if (navigator.sendBeacon) {
    navigator.sendBeacon('/analytics', body);
  } else {
    fetch('/analytics', { body, method: 'POST', keepalive: true });
  }
}

getCLS(sendToAnalytics);
getFID(sendToAnalytics);
getFCP(sendToAnalytics);
getLCP(sendToAnalytics);
getTTFB(sendToAnalytics);
```

### 2. React Profiler

```tsx
import { Profiler } from 'react';

const onRenderCallback = (
  id,
  phase,
  actualDuration,
  baseDuration,
  startTime,
  commitTime
) => {
  console.log(`${id} (${phase}) took ${actualDuration}ms`);
};

<Profiler id="Dashboard" onRender={onRenderCallback}>
  <Dashboard />
</Profiler>
```

### 3. Performance API

```tsx
// Mark and measure
performance.mark('component-render-start');
// ... component renders
performance.mark('component-render-end');

performance.measure(
  'component-render',
  'component-render-start',
  'component-render-end'
);

const measures = performance.getEntriesByName('component-render');
console.log(measures[0].duration);
```

## Best Practices

### ✅ Do

- Measure before optimizing
- Use code splitting
- Implement lazy loading
- Optimize images
- Cache API responses
- Use CDN for static assets
- Monitor Core Web Vitals
- Profile regularly

### ❌ Don't

- Premature optimization
- Ignore bundle size
- Load everything upfront
- Skip image optimization
- Forget cleanup in useEffect
- Ignore memory leaks
- Over-memoize

## Performance Budget

```javascript
// performance-budget.config.js
module.exports = {
  budgets: [
    {
      resourceSizes: [
        { resourceType: 'script', budget: 300 }, // 300KB
        { resourceType: 'style', budget: 50 },   // 50KB
        { resourceType: 'image', budget: 500 },  // 500KB
        { resourceType: 'total', budget: 1000 }, // 1MB total
      ],
      resourceCounts: [
        { resourceType: 'script', budget: 10 },
        { resourceType: 'stylesheet', budget: 5 },
      ],
    },
  ],
};
```

## Further Reading

- [Micro-Frontends](./micro-frontends.md)
- [Design Systems](./design-systems.md)
- [Frontend Patterns](../patterns/frontend-patterns.md)

---

*Next: [Micro-Frontends →](./micro-frontends.md)*

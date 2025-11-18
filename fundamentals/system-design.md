# System Design

## Introduction

System design is the process of defining the architecture, components, modules, interfaces, and data for a system to satisfy specified requirements. This guide covers key concepts and approaches for designing scalable, reliable systems.

## System Design Process

### 1. Requirements Gathering

#### Functional Requirements
What the system should do:
- User authentication
- Data storage and retrieval
- Business logic processing
- API endpoints

#### Non-Functional Requirements
How the system should perform:
- **Performance**: Response time, throughput
- **Scalability**: Handling growth
- **Availability**: Uptime requirements
- **Reliability**: Fault tolerance
- **Security**: Data protection
- **Maintainability**: Ease of updates

### 2. Capacity Estimation

```
Example: Design a URL shortener

Traffic Estimates:
- 100M URLs created per month
- 10:1 read to write ratio
- Write: 100M / (30 days * 24 hours * 3600s) ≈ 40 URLs/second
- Read: 400 URLs/second

Storage Estimates:
- Each URL: ~500 bytes
- Monthly storage: 100M * 500 bytes = 50 GB/month
- 5 years storage: 50 GB * 12 * 5 = 3 TB

Bandwidth Estimates:
- Write: 40 * 500 bytes = 20 KB/s
- Read: 400 * 500 bytes = 200 KB/s
```

## Key Concepts

### Scalability

#### Vertical Scaling (Scale Up)
Add more power to existing machines.

```
Before:           After:
┌──────────┐     ┌──────────┐
│ 4GB RAM  │  →  │ 16GB RAM │
│ 2 Cores  │     │ 8 Cores  │
└──────────┘     └──────────┘
```

**Pros:**
- Simple to implement
- No code changes needed
- Maintains data consistency

**Cons:**
- Hardware limits
- Single point of failure
- Expensive at scale

#### Horizontal Scaling (Scale Out)
Add more machines.

```
Before:                After:
┌──────────┐          ┌──────────┐ ┌──────────┐ ┌──────────┐
│ Server 1 │    →     │ Server 1 │ │ Server 2 │ │ Server 3 │
└──────────┘          └──────────┘ └──────────┘ └──────────┘
                              │
                      ┌───────┴────────┐
                      │ Load Balancer  │
                      └────────────────┘
```

**Pros:**
- No hardware limits
- Better fault tolerance
- Cost-effective

**Cons:**
- Complex implementation
- Data consistency challenges
- Network overhead

### Load Balancing

Distribute traffic across multiple servers.

```typescript
// Simple Round Robin Load Balancer
class LoadBalancer {
  private servers: string[] = [
    'server1.example.com',
    'server2.example.com',
    'server3.example.com'
  ];
  private currentIndex = 0;

  getNextServer(): string {
    const server = this.servers[this.currentIndex];
    this.currentIndex = (this.currentIndex + 1) % this.servers.length;
    return server;
  }
}

// Weighted Load Balancing
class WeightedLoadBalancer {
  private servers = [
    { url: 'server1.example.com', weight: 5 },
    { url: 'server2.example.com', weight: 3 },
    { url: 'server3.example.com', weight: 2 },
  ];

  getNextServer(): string {
    const totalWeight = this.servers.reduce((sum, s) => sum + s.weight, 0);
    let random = Math.random() * totalWeight;

    for (const server of this.servers) {
      random -= server.weight;
      if (random <= 0) return server.url;
    }

    return this.servers[0].url;
  }
}
```

**Load Balancing Algorithms:**
- **Round Robin**: Sequential distribution
- **Least Connections**: Send to server with fewest active connections
- **IP Hash**: Route based on client IP
- **Weighted**: Based on server capacity

### Caching

Store frequently accessed data in fast storage.

```
┌────────┐    Cache Miss    ┌───────┐
│ Client │ ──────────────── │ Cache │
└────┬───┘                  └───┬───┘
     │                          │
     │    Cache Miss            │
     └──────────────────────────┘
                │
         ┌──────▼──────┐
         │  Database   │
         └─────────────┘
```

#### Cache Strategies

**1. Cache-Aside (Lazy Loading)**
```typescript
async function getData(key: string) {
  // Try cache first
  let data = await cache.get(key);

  if (!data) {
    // Cache miss - get from database
    data = await database.get(key);
    // Store in cache
    await cache.set(key, data, TTL);
  }

  return data;
}
```

**2. Write-Through**
```typescript
async function saveData(key: string, value: any) {
  // Write to cache and database
  await cache.set(key, value);
  await database.save(key, value);
}
```

**3. Write-Behind (Write-Back)**
```typescript
async function saveData(key: string, value: any) {
  // Write to cache immediately
  await cache.set(key, value);

  // Write to database asynchronously
  queueDatabaseWrite(key, value);
}
```

**Cache Eviction Policies:**
- **LRU** (Least Recently Used)
- **LFU** (Least Frequently Used)
- **FIFO** (First In First Out)
- **TTL** (Time To Live)

### Database Design

#### SQL vs NoSQL

**SQL (Relational)**
```sql
-- Structured data with relationships
CREATE TABLE users (
  id INT PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100)
);

CREATE TABLE orders (
  id INT PRIMARY KEY,
  user_id INT REFERENCES users(id),
  total DECIMAL(10, 2)
);
```

**When to use SQL:**
- Complex queries and joins
- ACID compliance needed
- Structured data
- Strong consistency

**NoSQL (Document, Key-Value, etc.)**
```javascript
// Flexible schema
{
  "_id": "user123",
  "name": "John Doe",
  "email": "john@example.com",
  "orders": [
    { "id": "order1", "total": 100 },
    { "id": "order2", "total": 250 }
  ],
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}
```

**When to use NoSQL:**
- Flexible schema
- Horizontal scalability needed
- High write throughput
- Simple queries
- Document or key-value data

#### Database Sharding

Partition data across multiple databases.

```
Users Database - Sharded by User ID

Shard 1 (ID: 0-999):        Shard 2 (ID: 1000-1999):
┌───────────────┐           ┌───────────────┐
│ user_0        │           │ user_1000     │
│ user_1        │           │ user_1001     │
│ ...           │           │ ...           │
│ user_999      │           │ user_1999     │
└───────────────┘           └───────────────┘
```

```typescript
class ShardedDatabase {
  private shards: Database[];

  private getShardIndex(userId: string): number {
    // Hash-based sharding
    const hash = this.hashFunction(userId);
    return hash % this.shards.length;
  }

  async getUser(userId: string): Promise<User> {
    const shardIndex = this.getShardIndex(userId);
    return this.shards[shardIndex].query('SELECT * FROM users WHERE id = ?', [userId]);
  }
}
```

**Sharding Strategies:**
- **Range-based**: Partition by ID ranges
- **Hash-based**: Hash key to determine shard
- **Geographic**: Partition by location
- **Directory-based**: Lookup table for routing

### Database Replication

#### Master-Slave Replication

```
         ┌────────────┐
         │   Master   │ (Write)
         └──────┬─────┘
                │
       ┌────────┼────────┐
       │        │        │
  ┌────▼───┐ ┌─▼────┐ ┌─▼────┐
  │ Slave1 │ │Slave2│ │Slave3│ (Read)
  └────────┘ └──────┘ └──────┘
```

**Benefits:**
- Read scalability
- Backup/disaster recovery
- Offline processing

**Challenges:**
- Replication lag
- Consistency issues
- Master failure handling

#### Master-Master Replication

```
  ┌────────────┐     ┌────────────┐
  │  Master 1  │ ←──→│  Master 2  │
  └────────────┘     └────────────┘
```

**Benefits:**
- High availability
- Load distribution
- Geographic distribution

**Challenges:**
- Conflict resolution
- Complex synchronization
- Consistency management

### Message Queues

Asynchronous communication between services.

```
┌──────────┐     ┌───────────┐     ┌──────────┐
│ Producer │────→│   Queue   │────→│ Consumer │
└──────────┘     └───────────┘     └──────────┘
```

```typescript
// Producer
class OrderService {
  async createOrder(orderData: OrderData) {
    const order = await this.saveOrder(orderData);

    // Send to queue for async processing
    await messageQueue.publish('orders', {
      type: 'order.created',
      data: order
    });

    return order;
  }
}

// Consumer
class EmailWorker {
  async start() {
    messageQueue.subscribe('orders', async (message) => {
      if (message.type === 'order.created') {
        await this.sendOrderConfirmation(message.data);
      }
    });
  }
}
```

**Popular Message Queues:**
- RabbitMQ
- Apache Kafka
- AWS SQS
- Redis Pub/Sub

### CDN (Content Delivery Network)

Distribute static content geographically.

```
        ┌────────┐
        │ Client │
        └───┬────┘
            │
      ┌─────▼──────┐
      │ CDN (Edge) │
      └─────┬──────┘
            │
    ┌───────▼────────┐
    │ Origin Server  │
    └────────────────┘
```

**Benefits:**
- Reduced latency
- Lower bandwidth costs
- Improved availability
- DDoS protection

**What to cache:**
- Images, videos
- JavaScript, CSS
- Static HTML
- Fonts

## Design Patterns for Scale

### API Gateway

Single entry point for clients.

```
┌────────┐
│ Client │
└───┬────┘
    │
┌───▼──────────┐
│ API Gateway  │
└───┬──────────┘
    │
    ├──────────┬──────────┬──────────┐
    │          │          │          │
┌───▼───┐  ┌──▼───┐  ┌──▼───┐  ┌───▼───┐
│ Auth  │  │Users │  │Orders│  │Product│
└───────┘  └──────┘  └──────┘  └───────┘
```

```typescript
class APIGateway {
  async handleRequest(req: Request): Promise<Response> {
    // Authentication
    const user = await this.authenticate(req);

    // Rate limiting
    if (!this.checkRateLimit(user.id)) {
      return { status: 429, message: 'Too many requests' };
    }

    // Routing
    const service = this.getService(req.path);

    // Request transformation
    const transformedReq = this.transform(req);

    // Call service
    const response = await service.call(transformedReq);

    // Response transformation
    return this.transformResponse(response);
  }
}
```

### Circuit Breaker

Prevent cascading failures.

```typescript
class CircuitBreaker {
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';
  private failureCount = 0;
  private readonly threshold = 5;
  private readonly timeout = 60000; // 1 minute

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'OPEN') {
      if (Date.now() - this.openedAt > this.timeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess() {
    this.failureCount = 0;
    if (this.state === 'HALF_OPEN') {
      this.state = 'CLOSED';
    }
  }

  private onFailure() {
    this.failureCount++;
    if (this.failureCount >= this.threshold) {
      this.state = 'OPEN';
      this.openedAt = Date.now();
    }
  }
}

// Usage
const breaker = new CircuitBreaker();

async function callExternalService() {
  return breaker.execute(() => fetch('https://api.example.com/data'));
}
```

### Rate Limiting

Control request rate.

```typescript
class RateLimiter {
  private requests = new Map<string, number[]>();
  private readonly limit = 100; // requests
  private readonly window = 60000; // 1 minute

  isAllowed(userId: string): boolean {
    const now = Date.now();
    const userRequests = this.requests.get(userId) || [];

    // Remove old requests outside window
    const validRequests = userRequests.filter(
      time => now - time < this.window
    );

    if (validRequests.length >= this.limit) {
      return false;
    }

    validRequests.push(now);
    this.requests.set(userId, validRequests);
    return true;
  }
}
```

## Example: Design a URL Shortener

### Requirements

**Functional:**
- Generate short URL from long URL
- Redirect short URL to original
- Custom short URLs (optional)
- Analytics (clicks, referrers)

**Non-Functional:**
- Low latency (< 100ms)
- High availability (99.9%)
- Scalable (millions of URLs)

### High-Level Design

```
┌────────┐
│ Client │
└───┬────┘
    │
┌───▼────────────┐
│  Load Balancer │
└───┬────────────┘
    │
    ├──────────────┬──────────────┐
    │              │              │
┌───▼────┐    ┌───▼────┐    ┌───▼────┐
│ Web 1  │    │ Web 2  │    │ Web 3  │
└───┬────┘    └───┬────┘    └───┬────┘
    │             │             │
    └─────────┬───┴─────────────┘
              │
         ┌────▼─────┐
         │  Cache   │
         └────┬─────┘
              │
         ┌────▼─────┐
         │ Database │
         └──────────┘
```

### Database Schema

```sql
CREATE TABLE urls (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  short_code VARCHAR(10) UNIQUE NOT NULL,
  original_url TEXT NOT NULL,
  user_id BIGINT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  expires_at TIMESTAMP,
  INDEX idx_short_code (short_code)
);

CREATE TABLE analytics (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  short_code VARCHAR(10),
  clicked_at TIMESTAMP,
  referrer VARCHAR(255),
  user_agent VARCHAR(255),
  ip_address VARCHAR(45)
);
```

### API Design

```typescript
// POST /api/shorten
interface ShortenRequest {
  url: string;
  customAlias?: string;
  expiresAt?: Date;
}

interface ShortenResponse {
  shortUrl: string;
  shortCode: string;
}

// GET /:shortCode
// Redirects to original URL

// GET /api/stats/:shortCode
interface StatsResponse {
  totalClicks: number;
  clicksByDate: { date: string; clicks: number }[];
  topReferrers: { referrer: string; count: number }[];
}
```

## Best Practices

### ✅ Do

- Start with simple design
- Identify bottlenecks early
- Plan for failure
- Use proven patterns
- Document architecture
- Monitor everything
- Test at scale
- Plan for maintenance

### ❌ Don't

- Over-engineer initially
- Ignore non-functional requirements
- Forget about monitoring
- Skip capacity planning
- Assume infinite resources
- Ignore security
- Forget about data consistency

## Further Reading

- [Architecture Patterns](./patterns.md)
- [Architecture Principles](./principles.md)
- [Frontend Architecture](../frontend/fundamentals.md)

---

*Next: [Architecture Decision Records →](./adr.md)*

# NodeJS Interview Questions & Answers

## Core NodeJS Concepts

### 1. Event Loop & Asynchronous Programming

**Q: Explain the NodeJS Event Loop**

**Answer:**
The NodeJS Event Loop is the core mechanism that enables NodeJS to perform non-blocking I/O operations despite being single-threaded. It's a semi-infinite loop that continuously checks for and processes events.

**Key Components:**
- **Single Thread**: JavaScript runs on a single main thread
- **Non-blocking I/O**: I/O operations are offloaded to the system kernel
- **Event Queue**: Callbacks waiting to be executed
- **Call Stack**: Functions currently being executed

**Event Loop Phases (in order):**
1. **Timers**: Executes `setTimeout()` and `setInterval()` callbacks
2. **Pending Callbacks**: Executes I/O-related callbacks that were deferred
3. **Idle, Prepare**: Internal use only (used by NodeJS internally)
4. **Poll**: Retrieve new I/O events and execute their callbacks
5. **Check**: Executes `setImmediate()` callbacks
6. **Close Callbacks**: Executes `close` event callbacks

**Why it matters:**
- Enables high concurrency with minimal overhead
- Prevents blocking the main thread during I/O operations
- Allows handling thousands of simultaneous connections

**Common Pitfalls:**
- Heavy computation blocks the event loop
- Unhandled promise rejections can crash the application
- Microtasks (promises) have higher priority than macrotasks (setTimeout)

```javascript
// Event Loop demonstration
console.log('Start');

setTimeout(() => {
  console.log('Timer callback');
}, 0);

Promise.resolve().then(() => {
  console.log('Promise callback');
});

console.log('End');
// Output: Start, End, Promise callback, Timer callback
```

**Q: What's the difference between `process.nextTick()` and `setImmediate()`?**

**Answer:**
Both functions schedule callbacks to be executed asynchronously, but they have different timing and priority in the event loop.

**`process.nextTick()`:**
- **Timing**: Executes immediately after the current operation completes, before the next event loop iteration
- **Priority**: Higher than any other asynchronous operation
- **Use Case**: When you need to ensure code runs after the current synchronous code but before any I/O events
- **Risk**: Can cause I/O starvation if used excessively (preventing event loop from progressing)

**`setImmediate()`:**
- **Timing**: Executes in the "check" phase of the event loop
- **Priority**: Lower than `nextTick()`, runs after I/O events
- **Use Case**: When you want to execute code after I/O operations have been processed
- **Safety**: Won't block I/O operations

**Execution Order:**
```javascript
console.log('Start');

process.nextTick(() => console.log('nextTick'));
setImmediate(() => console.log('setImmediate'));

console.log('End');
// Output: Start, End, nextTick, setImmediate
```

**Best Practices:**
- Use `setImmediate()` for most cases to avoid I/O starvation
- Use `nextTick()` only when you need to break up long-running synchronous operations
- Be cautious with recursive `nextTick()` calls as they can block the event loop

### 2. Modules & Dependency Management

**Q: CommonJS vs ES Modules**

**Answer:**
NodeJS supports two module systems: CommonJS (the original) and ES Modules (the modern standard). They have different syntax, behavior, and use cases.

**CommonJS (CJS):**
- **Syntax**: `require()` for imports, `module.exports` for exports
- **Loading**: Synchronous (blocks execution until module loads)
- **Resolution**: NodeJS-specific algorithm
- **File Extensions**: `.js` (default), `.cjs` (explicit)
- **Features**: Dynamic imports, `__dirname`, `__filename`
- **Use Case**: Traditional NodeJS applications, legacy code

**ES Modules (ESM):**
- **Syntax**: `import`/`export` statements
- **Loading**: Asynchronous (non-blocking)
- **Resolution**: Browser-compatible, supports tree-shaking
- **File Extensions**: `.mjs` (explicit), `.js` (with `"type": "module"` in package.json)
- **Features**: Static analysis, better optimization, top-level await
- **Use Case**: Modern applications, frontend compatibility, better tooling

**Key Differences:**

| Feature | CommonJS | ES Modules |
|---------|----------|------------|
| Loading | Synchronous | Asynchronous |
| Static Analysis | Limited | Full support |
| Tree Shaking | Not supported | Supported |
| Dynamic Imports | `require()` anywhere | `import()` only |
| Caching | Runtime cache | Compile-time cache |

**Interoperability:**
```javascript
// In CommonJS, importing ES Module
const esModule = require('./es-module.mjs');

// In ES Module, importing CommonJS
import cjsModule from './cjs-module.cjs';
```

**Migration Strategy:**
- Use `.mjs` extension for new ES modules
- Use `.cjs` for CommonJS when mixing both
- Set `"type": "module"` in package.json for ES module defaults
- Consider using build tools (webpack, rollup) for complex projects

```javascript
// CommonJS
const fs = require('fs');
module.exports = { myFunction };

// ES Modules
import fs from 'fs';
export { myFunction };
```

**Q: What is the `package.json` file?**

**Answer:**
The `package.json` file is the heart of any NodeJS project. It's a JSON file that contains metadata about the project, its dependencies, scripts, and configuration.

**Key Sections:**

**1. Project Metadata:**
```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "A sample NodeJS application",
  "main": "index.js",
  "author": "John Doe",
  "license": "MIT"
}
```
- **name**: Unique identifier for the package
- **version**: Semantic versioning (major.minor.patch)
- **main**: Entry point of the application

**2. Dependencies:**
```json
{
  "dependencies": {
    "express": "^4.18.0",
    "mongoose": "~6.0.0"
  },
  "devDependencies": {
    "jest": "^27.0.0",
    "nodemon": "^2.0.0"
  }
}
```
- **dependencies**: Required for production runtime
- **devDependencies**: Only needed for development/testing
- **Version Ranges**: `^` (compatible), `~` (patch updates), `*` (latest)

**3. Scripts:**
```json
{
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest",
    "build": "webpack --mode production"
  }
}
```
- Predefined scripts: `npm start`, `npm test`, `npm stop`
- Custom scripts: `npm run dev`, `npm run build`

**4. Configuration:**
```json
{
  "engines": {
    "node": ">=14.0.0",
    "npm": ">=6.0.0"
  },
  "type": "module",
  "private": true
}
```
- **engines**: Specify required NodeJS/npm versions
- **type**: Set to "module" for ES modules
- **private**: Prevent accidental publishing

**Important Commands:**
- `npm install`: Install all dependencies
- `npm install <package>`: Add new dependency
- `npm install <package> --save-dev`: Add dev dependency
- `npm update`: Update dependencies
- `npm audit`: Security vulnerability check

**Best Practices:**
- Use semantic versioning
- Keep dependencies minimal
- Regular security audits with `npm audit`
- Use `package-lock.json` for reproducible builds
- Specify engine versions for deployment consistency

### 3. Streams & Buffers

**Q: Explain NodeJS Streams**

**Answer:**
Streams are a fundamental abstraction in NodeJS for handling data flow. They allow you to read from or write to data sources in a continuous, chunk-by-chunk manner rather than loading everything into memory at once.

**Why Streams Matter:**
- **Memory Efficiency**: Process large files without loading entire content into RAM
- **Time Efficiency**: Start processing data as soon as it's available
- **Composability**: Chain multiple operations together
- **Backpressure**: Automatically handle speed differences between producer and consumer

**Four Stream Types:**

1. **Readable Streams**: Source of data that can be read from
   - Examples: `fs.createReadStream()`, HTTP requests, process.stdin
   - Events: `data`, `end`, `error`, `close`

2. **Writable Streams**: Destination for data that can be written to
   - Examples: `fs.createWriteStream()`, HTTP responses, process.stdout
   - Events: `drain`, `finish`, `error`, `close`

3. **Duplex Streams**: Both readable and writable
   - Examples: TCP sockets, zlib streams
   - Can read and write independently

4. **Transform Streams**: Duplex streams that modify data as it passes through
   - Examples: zlib compression, crypto streams
   - Transform data during read/write operations

**Stream Modes:**
- **Flowing Mode**: Data flows automatically as soon as available
- **Paused Mode**: Data must be explicitly requested with `read()`

**Common Stream Operations:**
```javascript
const fs = require('fs');
const zlib = require('zlib');

// Basic piping
const readable = fs.createReadStream('input.txt');
const writable = fs.createWriteStream('output.txt');
readable.pipe(writable);

// Chaining operations (compression)
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('output.txt.gz'));

// Manual stream handling
const readable = fs.createReadStream('input.txt');
readable.on('data', (chunk) => {
  console.log(`Received ${chunk.length} bytes`);
});
readable.on('end', () => {
  console.log('Finished reading');
});
```

**Stream Implementation Example:**
```javascript
class UppercaseTransform extends require('stream').Transform {
  constructor() {
    super();
  }

  _transform(chunk, encoding, callback) {
    const uppercased = chunk.toString().toUpperCase();
    callback(null, uppercased);
  }
}

// Usage
fs.createReadStream('input.txt')
  .pipe(new UppercaseTransform())
  .pipe(fs.createWriteStream('output.txt'));
```

**Best Practices:**
- Use streams for large files to prevent memory issues
- Always handle error events on streams
- Use `pipe()` for simple stream connections
- Implement backpressure handling for custom streams
- Consider using libraries like `pipeline()` for complex stream operations

```javascript
const fs = require('fs');
const readable = fs.createReadStream('input.txt');
const writable = fs.createWriteStream('output.txt');

readable.pipe(writable);
```

**Q: What are Buffers?**

**Answer:**
Buffers in NodeJS are fixed-size memory allocations used to handle binary data. Since JavaScript traditionally doesn't have a native way to work with raw binary data, NodeJS introduced the Buffer class to fill this gap.

**Why Buffers Are Needed:**
- **Binary Data Handling**: Work with files, network protocols, cryptography
- **Performance**: More efficient than strings for binary operations
- **Memory Management**: Fixed-size allocations prevent memory fragmentation
- **Interoperability**: Interface with C++ addons and system APIs

**Key Characteristics:**
- **Global Available**: No need to import `Buffer` class
- **Fixed Size**: Cannot be resized after creation
- **Raw Memory**: Direct memory access for performance
- **Encoding Support**: Multiple text encoding formats

**Creating Buffers:**
```javascript
// From string
const buf1 = Buffer.from('Hello World', 'utf8');

// From array
const buf2 = Buffer.from([0x48, 0x65, 0x6c, 0x6c, 0x6f]);

// Empty buffer of specific size
const buf3 = Buffer.alloc(1024); // 1KB of zeroed memory

// Unsafe allocation (faster but contains old memory)
const buf4 = Buffer.allocUnsafe(1024);
```

**Common Operations:**
```javascript
const buf = Buffer.from('Hello World');

// Convert to string
console.log(buf.toString()); // 'Hello World'
console.log(buf.toString('hex')); // '48656c6c6f20576f726c64'
console.log(buf.toString('base64')); // 'SGVsbG8gV29ybGQ='

// Get buffer length
console.log(buf.length); // 11

// Read/write individual bytes
console.log(buf[0]); // 72 (ASCII code for 'H')
buf[0] = 74; // Change to 'J'

// Slice buffer (creates view, not copy)
const slice = buf.slice(0, 5);

// Copy buffer
const copy = Buffer.alloc(buf.length);
buf.copy(copy);

// Concatenate buffers
const concatenated = Buffer.concat([buf1, buf2]);
```

**Encoding Support:**
- **utf8**: Default, variable-length Unicode encoding
- **ascii**: 7-bit ASCII characters
- **utf16le**: 2-byte little-endian Unicode
- **base64**: Base64 encoding
- **hex**: Hexadecimal representation
- **binary**: Legacy (use utf8 instead)

**Buffer vs Array:**
```javascript
// Buffer (efficient for binary data)
const buf = Buffer.from([0x48, 0x65, 0x6c, 0x6c, 0x6f]);
console.log(buf.toString()); // 'Hello'

// Regular Array (inefficient for binary)
const arr = [72, 101, 108, 108, 111];
console.log(String.fromCharCode(...arr)); // 'Hello'
```

**Use Cases:**
- **File I/O**: Reading/writing binary files
- **Network Programming**: TCP/UDP packet handling
- **Cryptography**: Encryption/decryption operations
- **Image Processing**: Working with image data
- **Database**: Storing binary data (BLOB)

**Best Practices:**
- Use `Buffer.alloc()` instead of `Buffer.allocUnsafe()` for security
- Be aware of encoding when converting to/from strings
- Use buffers for large binary data operations
- Remember buffers are mutable - copy if you need immutability
- Consider using `Uint8Array` for better browser compatibility

```javascript
const buf = Buffer.from('Hello World', 'utf8');
console.log(buf.toString('hex')); // 48656c6c6f20576f726c64
```

## Advanced NodeJS Topics

### 4. Performance & Optimization

**Q: How would you optimize NodeJS performance?**

**Answer:**
Optimizing NodeJS performance involves multiple layers of improvements, from code-level optimizations to infrastructure changes. Here's a comprehensive approach:

**1. Code-Level Optimizations:**

**Event Loop Optimization:**
- Avoid blocking operations in the main thread
- Use worker threads for CPU-intensive tasks
- Break up long-running synchronous operations

```javascript
// Bad: Blocking operation
function heavyComputation() {
  for (let i = 0; i < 1000000000; i++) {
    // Complex calculation
  }
}

// Good: Use worker threads
const { Worker } = require('worker_threads');
function heavyComputation() {
  return new Promise((resolve) => {
    const worker = new Worker('./heavy-task.js');
    worker.on('message', resolve);
  });
}
```

**Memory Management:**
- Avoid memory leaks by cleaning up event listeners
- Use object pooling for frequently created objects
- Monitor heap usage and garbage collection

**2. Database Optimizations:**

**Connection Pooling:**
```javascript
const mysql = require('mysql2/promise');
const pool = mysql.createPool({
  host: 'localhost',
  user: 'user',
  password: 'password',
  database: 'db',
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});
```

**Query Optimization:**
- Use indexes effectively
- Implement query result caching
- Use prepared statements for repeated queries
- Avoid N+1 query problems

**3. Caching Strategies:**

**Multi-Level Caching:**
```javascript
const redis = require('redis');
const client = redis.createClient();

class CacheService {
  static async get(key) {
    // Try memory cache first
    if (this.memoryCache[key]) {
      return this.memoryCache[key];
    }
    
    // Try Redis cache
    const cached = await client.get(key);
    if (cached) {
      this.memoryCache[key] = JSON.parse(cached);
      return this.memoryCache[key];
    }
    
    return null;
  }
  
  static async set(key, value, ttl = 3600) {
    this.memoryCache[key] = value;
    await client.setex(key, ttl, JSON.stringify(value));
  }
}
```

**4. HTTP Optimizations:**

**Compression:**
```javascript
const compression = require('compression');
app.use(compression({
  level: 6,
  threshold: 1024
}));
```

**Response Streaming:**
```javascript
app.get('/large-file', (req, res) => {
  const fileStream = fs.createReadStream('./large-file.json');
  fileStream.pipe(res);
});
```

**5. Cluster Module for Multi-Core:**

```javascript
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);
  
  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // Restart worker
  });
} else {
  // Worker processes can share any TCP port
  require('./server');
  console.log(`Worker ${process.pid} started`);
}
```

**6. Monitoring and Profiling:**

**Performance Monitoring:**
```javascript
const { performance, PerformanceObserver } = require('perf_hooks');

const obs = new PerformanceObserver((list) => {
  const entries = list.getEntries();
  entries.forEach((entry) => {
    console.log(`${entry.name}: ${entry.duration}ms`);
  });
});
obs.observe({ entryTypes: ['measure'] });

// Usage
performance.mark('start');
// ... some operation
performance.mark('end');
performance.measure('operation', 'start', 'end');
```

**7. Infrastructure Optimizations:**

**Load Balancing:**
- Use Nginx or HAProxy for load balancing
- Implement health checks
- Configure sticky sessions if needed

**CDN Integration:**
- Serve static assets from CDN
- Implement edge caching
- Use geographic distribution

**8. Best Practices Summary:**
- **Profile First**: Measure before optimizing
- **Async Everywhere**: Never block the event loop
- **Cache Strategically**: Cache frequently accessed data
- **Monitor Continuously**: Track performance metrics
- **Scale Horizontally**: Use multiple processes/instances
- **Optimize Database**: Most performance issues are database-related
- **Use Compression**: Reduce network payload size
- **Implement Rate Limiting**: Prevent abuse and overload

```javascript
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
} else {
  // Worker processes
  require('./server');
}
```

**Q: Memory management in NodeJS**

**Answer:**
Memory management in NodeJS is crucial for building performant and reliable applications. Understanding how V8 handles memory and common pitfalls helps prevent memory leaks and optimize performance.

**V8 Memory Architecture:**

**Heap Structure:**
- **New Space**: Young generation for short-lived objects (scavenger GC)
- **Old Space**: Old generation for long-lived objects (mark-sweep GC)
- **Code Space**: Executable code
- **Large Object Space**: Objects > 1MB
- **Other Spaces**: Map space, cell space, property cell space

**Garbage Collection:**
1. **Scavenger (Minor GC)**: Fast, frequent, handles new space
2. **Mark-Sweep (Major GC)**: Slower, handles old space
3. **Incremental Marking**: Spread GC work over multiple frames
4. **Idle-time GC**: Run during application idle periods

**Memory Limits:**
```javascript
// Default limits (can be increased)
// 32-bit: ~1.4GB heap
// 64-bit: ~1.7GB heap

// Increase memory limit
node --max-old-space-size=4096 app.js // 4GB
```

**Common Memory Leaks:**

**1. Global Variables:**
```javascript
// Bad: Global variable grows indefinitely
let globalCache = {};

function addToCache(key, value) {
  globalCache[key] = value; // Never cleared
}

// Good: Use WeakMap or implement cleanup
const cache = new Map();
function addToCache(key, value) {
  if (cache.size > 1000) {
    const firstKey = cache.keys().next().value;
    cache.delete(firstKey);
  }
  cache.set(key, value);
}
```

**2. Event Listeners:**
```javascript
// Bad: Event listeners never removed
const emitter = new EventEmitter();

function setupListeners() {
  emitter.on('data', handleData);
  // Listeners accumulate on repeated calls
}

// Good: Clean up listeners
function setupListeners() {
  const handleData = (data) => { /* ... */ };
  emitter.on('data', handleData);
  
  return () => {
    emitter.off('data', handleData);
  };
}
```

**3. Closures:**
```javascript
// Bad: Closure retains large object
function createHandler() {
  const largeObject = new Array(1000000).fill(0);
  
  return function() {
    console.log('Handler called');
    // largeObject is kept in memory even if not used
  };
}

// Good: Minimize closure scope
function createHandler() {
  return function() {
    console.log('Handler called');
  };
}
```

**Memory Profiling Tools:**

**1. Heap Snapshots:**
```javascript
const heapdump = require('heapdump');

// Generate heap snapshot
heapdump.writeSnapshot('./heap-' + Date.now() + '.heapsnapshot');

// Programmatic heap analysis
const v8 = require('v8');
const heapStats = v8.getHeapStatistics();
console.log('Heap stats:', heapStats);
```

**2. Memory Usage Monitoring:**
```javascript
function monitorMemory() {
  const used = process.memoryUsage();
  
  console.log('Memory Usage:');
  for (let key in used) {
    console.log(`${key}: ${Math.round(used[key] / 1024 / 1024 * 100) / 100} MB`);
  }
  
  // Alert if memory usage is high
  if (used.heapUsed > 500 * 1024 * 1024) { // 500MB
    console.warn('High memory usage detected!');
  }
}

setInterval(monitorMemory, 5000);
```

**3. Chrome DevTools:**
```bash
# Start with debugging
node --inspect app.js

# Open Chrome DevTools and connect to localhost:9229
# Use Memory tab for heap snapshots and profiling
```

**Optimization Strategies:**

**1. Object Pooling:**
```javascript
class ObjectPool {
  constructor(createFn, resetFn, maxSize = 100) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    this.pool = [];
    this.maxSize = maxSize;
  }
  
  acquire() {
    if (this.pool.length > 0) {
      return this.pool.pop();
    }
    return this.createFn();
  }
  
  release(obj) {
    if (this.pool.length < this.maxSize) {
      this.resetFn(obj);
      this.pool.push(obj);
    }
  }
}

// Usage
const bufferPool = new ObjectPool(
  () => Buffer.alloc(1024),
  (buf) => buf.fill(0),
  50
);
```

**2. Stream Processing:**
```javascript
// Bad: Load entire file into memory
function processLargeFile(filename) {
  const data = fs.readFileSync(filename); // Memory intensive
  // Process data
}

// Good: Stream processing
function processLargeFile(filename) {
  const stream = fs.createReadStream(filename);
  stream.on('data', (chunk) => {
    // Process chunk incrementally
  });
}
```

**3. Weak References:**
```javascript
const WeakMap = require('weak-map');

// Use WeakMap for cache that doesn't prevent GC
const cache = new WeakMap();

function cacheResult(obj, result) {
  cache.set(obj, result);
  // obj can be garbage collected, cache entry will be removed
}
```

**Best Practices:**
- **Monitor Regularly**: Track memory usage trends
- **Profile Memory**: Use heap snapshots to identify leaks
- **Clean Up Resources**: Remove event listeners, close connections
- **Use Streams**: Process large data incrementally
- **Limit Caches**: Implement size limits and expiration
- **Avoid Globals**: Use module-scoped variables instead
- **Test Under Load**: Memory issues often appear under stress

**Common Memory Issues and Solutions:**
| Issue | Cause | Solution |
|-------|-------|----------|
| Memory leak | Unclosed connections, event listeners | Proper cleanup, weak references |
| High heap usage | Large objects, inefficient algorithms | Object pooling, streaming |
| GC pressure | Frequent object creation | Reuse objects, reduce allocations |
| Out of memory | Exceeding heap limits | Increase limits, optimize usage |

### 5. Error Handling

**Q: Error handling patterns in NodeJS**
- **Try-catch**: Synchronous code
- **Error-first callbacks**: `function(err, data) {}`
- **Promises**: `.catch()` method
- **Async/await**: Try-catch with async functions

```javascript
// Error-first callback
fs.readFile('file.txt', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Promise
fs.promises.readFile('file.txt')
  .then(data => console.log(data))
  .catch(err => console.error(err));

// Async/await
async function readFile() {
  try {
    const data = await fs.promises.readFile('file.txt');
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}
```

**Q: Unhandled promise rejections**
- **Process events**: `unhandledRejection`, `uncaughtException`
- **Best practices**: Always handle rejections
- **Graceful shutdown**: Clean up resources

### 6. Security

**Q: Security best practices in NodeJS**
- **Input validation**: `joi`, `express-validator`
- **Authentication**: JWT, Passport.js
- **HTTPS**: TLS/SSL encryption
- **Environment variables**: `.env` files
- **Dependencies**: Regular security audits

```javascript
const helmet = require('helmet');
app.use(helmet()); // Security headers

const rateLimit = require('express-rate-limit');
app.use(rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests
}));
```

**Q: Preventing common vulnerabilities**
- **SQL Injection**: Parameterized queries, ORMs
- **XSS**: Input sanitization, output encoding
- **CSRF**: CSRF tokens
- **DoS**: Rate limiting, request validation

## Framework & Ecosystem

### 7. Express.js

**Q: Express.js middleware**
- **Request pipeline**: Functions that process requests
- **Types**: Application-level, router-level, error-handling
- **Order matters**: Middleware executes in sequence

```javascript
app.use((req, res, next) => {
  console.log('Request received');
  next();
});

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

**Q: RESTful API design with Express**
- **HTTP methods**: GET, POST, PUT, DELETE
- **Status codes**: 200, 201, 400, 404, 500
- **Routing**: Parameterized routes, middleware

### 8. Database Integration

**Q: Working with databases in NodeJS**
- **SQL**: Sequelize, Knex.js, pg
- **NoSQL**: Mongoose (MongoDB), Redis
- **Connection pooling**: Manage database connections
- **ORM vs Query Builder**: Trade-offs

```javascript
// Mongoose example
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const userSchema = new Schema({
  name: String,
  email: { type: String, unique: true }
});

const User = mongoose.model('User', userSchema);
```

**Q: Database transactions**
- **ACID properties**: Atomicity, Consistency, Isolation, Durability
- **Implementation**: `sequelize.transaction()`, MongoDB sessions
- **Error handling**: Rollback on failure

### 9. Testing

**Q: Testing in NodeJS**
- **Frameworks**: Jest, Mocha, Chai
- **Types**: Unit, integration, end-to-end
- **Mocking**: `jest.mock()`, `sinon`
- **Coverage**: Code coverage reports

```javascript
// Jest example
describe('UserService', () => {
  test('should create user', async () => {
    const user = await UserService.create({
      name: 'John',
      email: 'john@example.com'
    });
    
    expect(user.name).toBe('John');
    expect(user.id).toBeDefined();
  });
});
```

**Q: Test-driven development**
- **Red-Green-Refactor**: Write failing test, make it pass, refactor
- **Benefits**: Better design, regression protection
- **Challenges**: Initial time investment

## Real-World Scenarios

### 10. Scaling & Architecture

**Q: How would you scale a NodeJS application?**
- **Horizontal scaling**: Load balancer, multiple instances
- **Vertical scaling**: More CPU, memory
- **Microservices**: Split into smaller services
- **Caching**: Redis, CDN

**Q: Microservices with NodeJS**
- **Communication**: REST APIs, message queues
- **Service discovery**: Consul, etcd
- **API Gateway**: Express Gateway, Kong
- **Monitoring**: Prometheus, Grafana

### 11. DevOps & Deployment

**Q: Deployment strategies**
- **PM2**: Process manager for NodeJS
- **Docker**: Containerization
- **CI/CD**: GitHub Actions, Jenkins
- **Environment management**: Development, staging, production

```javascript
// PM2 ecosystem file
module.exports = {
  apps: [{
    name: 'api-server',
    script: './server.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production'
    }
  }]
};
```

**Q: Monitoring and logging**
- **Winston**: Structured logging
- **Morgan**: HTTP request logging
- **APM**: New Relic, DataDog
- **Health checks**: `/health` endpoints

### 12. Advanced Patterns

**Q: Design patterns in NodeJS**
- **Singleton**: Database connections
- **Factory**: Object creation
- **Observer**: Event-driven programming
- **Strategy**: Algorithm selection

**Q: Event-driven architecture**
- **EventEmitter**: Core NodeJS pattern
- **Pub/Sub**: Redis pub/sub, message queues
- **CQRS**: Command Query Responsibility Segregation
- **Event sourcing**: Immutable event log

```javascript
const EventEmitter = require('events');

class UserService extends EventEmitter {
  async createUser(userData) {
    const user = await User.create(userData);
    this.emit('userCreated', user);
    return user;
  }
}

const userService = new UserService();
userService.on('userCreated', (user) => {
  console.log(`New user created: ${user.id}`);
});
```

## Practical Coding Questions

### 13. Algorithm Implementations

**Q: Implement a simple HTTP server**
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  if (req.method === 'GET' && req.url === '/') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello World');
  } else {
    res.writeHead(404, { 'Content-Type': 'text/plain' });
    res.end('Not Found');
  }
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

**Q: Implement a rate limiter**
```javascript
class RateLimiter {
  constructor(maxRequests, windowMs) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
    this.requests = new Map();
  }

  isAllowed(ip) {
    const now = Date.now();
    const windowStart = now - this.windowMs;
    
    if (!this.requests.has(ip)) {
      this.requests.set(ip, []);
    }
    
    const ipRequests = this.requests.get(ip);
    // Remove old requests
    const validRequests = ipRequests.filter(time => time > windowStart);
    
    if (validRequests.length >= this.maxRequests) {
      return false;
    }
    
    validRequests.push(now);
    this.requests.set(ip, validRequests);
    return true;
  }
}
```

**Q: Implement a simple cache**
```javascript
class Cache {
  constructor(ttl = 60000) {
    this.cache = new Map();
    this.ttl = ttl;
  }

  set(key, value) {
    this.cache.set(key, {
      value,
      timestamp: Date.now()
    });
  }

  get(key) {
    const item = this.cache.get(key);
    if (!item) return null;
    
    if (Date.now() - item.timestamp > this.ttl) {
      this.cache.delete(key);
      return null;
    }
    
    return item.value;
  }

  clear() {
    this.cache.clear();
  }
}
```

## Debugging & Troubleshooting

### 14. Common Issues

**Q: How to debug NodeJS applications?**
- **Node Inspector**: `node --inspect`
- **VS Code Debugger**: Built-in debugging
- **Console logging**: `console.log`, `console.error`
- **Memory profiling**: Heap snapshots

**Q: Common performance issues**
- **Blocking operations**: Synchronous I/O
- **Memory leaks**: Unreleased resources
- **CPU intensive tasks**: Blocking the event loop
- **Database queries**: N+1 query problem

**Q: Handling high concurrency**
- **Worker threads**: CPU-intensive tasks
- **Connection pooling**: Database connections
- **Queues**: Background job processing
- **Load balancing**: Distribute requests

## Best Practices & Tips

### 15. Code Quality

**Q: Code organization best practices**
- **Separation of concerns**: Routes, controllers, services
- **Dependency injection**: Testable code
- **Configuration management**: Environment-based
- **Error handling**: Consistent error responses

**Q: Security checklist**
- **Input validation**: Always validate user input
- **Authentication**: Strong password policies
- **Authorization**: Role-based access control
- **Dependencies**: Regular security updates

### 16. Interview Preparation Tips

**Technical Preparation:**
1. **Understand the event loop** thoroughly
2. **Practice coding problems** on platforms like LeetCode
3. **Build projects** using NodeJS
4. **Review common patterns** and anti-patterns

**During the Interview:**
1. **Think out loud** when solving problems
2. **Ask clarifying questions** about requirements
3. **Consider trade-offs** in your solutions
4. **Discuss testing** and error handling

**Common Follow-up Questions:**
- "How would you test this solution?"
- "What are the security implications?"
- "How would this scale?"
- "What are the performance characteristics?"

## Resources for Further Learning

### Documentation & Guides
- **Official NodeJS Documentation**: nodejs.org/docs
- **NodeJS Best Practices**: github.com/goldbergyoni/nodebestpractices
- **ExpressJS Guide**: expressjs.com/en/guide

### Books
- "Node.js Design Patterns" by Mario Casciaro
- "Node.js in Action" by Mike Cantelon
- "Professional Node.js" by Jonathan Leeser

### Online Courses
- NodeJS courses on Udemy, Coursera
- FreeCodeCamp NodeJS curriculum
- Node University (Azat Mardan)

### Practice Projects
- REST API with Express and MongoDB
- Real-time chat application with Socket.io
- File processing service with streams
- Microservices architecture with Docker

---

**Remember**: The key to NodeJS interviews is understanding the asynchronous nature of the platform and being able to explain the trade-offs of different architectural decisions. Good luck!

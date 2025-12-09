# Performance & Scaling trong Node.js

## 1. Clustering

Node.js chạy single-threaded, clustering cho phép tận dụng multi-core CPU.

```javascript
const cluster = require("cluster");
const http = require("http");
const numCPUs = require("os").cpus().length;

if (cluster.isPrimary) {
  console.log(`Primary ${process.pid} is running`);

  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on("exit", (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // Restart worker
  });
} else {
  http
    .createServer((req, res) => {
      res.writeHead(200);
      res.end("Hello World\n");
    })
    .listen(8000);

  console.log(`Worker ${process.pid} started`);
}
```

### PM2 (Production Process Manager)

```bash
npm install pm2 -g

# Start with cluster mode
pm2 start app.js -i max  # max = số CPU cores
pm2 start app.js -i 4    # 4 instances

# Commands
pm2 list
pm2 logs
pm2 monit
pm2 reload app    # Zero-downtime reload
pm2 stop app
pm2 delete app

# Ecosystem file
pm2 ecosystem
```

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: "app",
      script: "./app.js",
      instances: "max",
      exec_mode: "cluster",
      env: {
        NODE_ENV: "development",
      },
      env_production: {
        NODE_ENV: "production",
      },
    },
  ],
};
```

## 2. Worker Threads

Cho CPU-intensive tasks (không block event loop).

```javascript
const { Worker, isMainThread, parentPort, workerData } = require("worker_threads");

if (isMainThread) {
  // Main thread
  const worker = new Worker(__filename, {
    workerData: { num: 42 },
  });

  worker.on("message", (result) => {
    console.log("Result:", result);
  });

  worker.on("error", (err) => {
    console.error("Worker error:", err);
  });

  worker.on("exit", (code) => {
    console.log("Worker exited with code:", code);
  });
} else {
  // Worker thread
  const { num } = workerData;

  // CPU-intensive task
  const result = heavyComputation(num);

  parentPort.postMessage(result);
}
```

### Worker Pool

```javascript
const { StaticPool } = require("node-worker-threads-pool");

const pool = new StaticPool({
  size: 4,
  task: "./worker.js",
});

// Use pool
const result = await pool.exec({ data: "input" });
```

## 3. Caching Strategies

### In-Memory Cache (node-cache)

```javascript
const NodeCache = require("node-cache");
const cache = new NodeCache({ stdTTL: 600 }); // 10 minutes

async function getUser(id) {
  const cacheKey = `user:${id}`;

  // Check cache
  const cached = cache.get(cacheKey);
  if (cached) return cached;

  // Fetch from DB
  const user = await User.findById(id);

  // Store in cache
  cache.set(cacheKey, user);

  return user;
}
```

### Redis Cache

```javascript
const Redis = require("ioredis");
const redis = new Redis();

async function getUser(id) {
  const cacheKey = `user:${id}`;

  // Check cache
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);

  // Fetch from DB
  const user = await User.findById(id);

  // Store in cache (with TTL)
  await redis.setex(cacheKey, 600, JSON.stringify(user));

  return user;
}

// Cache invalidation
async function updateUser(id, data) {
  await User.findByIdAndUpdate(id, data);
  await redis.del(`user:${id}`);
}
```

### Cache Patterns

```javascript
// Cache-Aside (Lazy Loading)
async function getData(key) {
  let data = await cache.get(key);
  if (!data) {
    data = await db.get(key);
    await cache.set(key, data);
  }
  return data;
}

// Write-Through
async function setData(key, value) {
  await db.set(key, value);
  await cache.set(key, value);
}

// Write-Behind (Async)
async function setData(key, value) {
  await cache.set(key, value);
  queue.add({ key, value }); // Process later
}
```

## 4. Database Optimization

### Connection Pooling

```javascript
// PostgreSQL
const { Pool } = require("pg");
const pool = new Pool({
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// MongoDB
mongoose.connect(uri, {
  maxPoolSize: 10,
  minPoolSize: 5,
  serverSelectionTimeoutMS: 5000,
});
```

### Query Optimization

```javascript
// ❌ N+1 Problem
const users = await User.find();
for (const user of users) {
  const posts = await Post.find({ authorId: user.id }); // N queries!
}

// ✅ Eager Loading
const users = await User.find().populate("posts"); // 2 queries

// ✅ Batch Loading
const userIds = users.map((u) => u.id);
const posts = await Post.find({ authorId: { $in: userIds } });
```

### Indexing

```javascript
// MongoDB
userSchema.index({ email: 1 }, { unique: true });
userSchema.index({ createdAt: -1 });
userSchema.index({ name: "text", bio: "text" }); // Text search

// Compound index
userSchema.index({ status: 1, createdAt: -1 });
```

## 5. Memory Management

### Detect Memory Leaks

```javascript
// Check memory usage
const used = process.memoryUsage();
console.log({
  rss: `${Math.round(used.rss / 1024 / 1024)} MB`,
  heapTotal: `${Math.round(used.heapTotal / 1024 / 1024)} MB`,
  heapUsed: `${Math.round(used.heapUsed / 1024 / 1024)} MB`,
  external: `${Math.round(used.external / 1024 / 1024)} MB`,
});

// Heap snapshot
const v8 = require("v8");
v8.writeHeapSnapshot();
```

### Common Memory Leaks

```javascript
// ❌ Global variables
global.cache = []; // Grows forever

// ❌ Event listeners not removed
emitter.on("event", handler);
// Fix: emitter.off('event', handler);

// ❌ Closures holding references
function createLeak() {
  const bigData = new Array(1000000);
  return () => bigData.length; // bigData never GC'd
}

// ❌ Timers not cleared
const interval = setInterval(() => {}, 1000);
// Fix: clearInterval(interval);
```

## 6. Streams for Large Data

```javascript
const fs = require("fs");
const { Transform } = require("stream");

// ❌ Bad - loads entire file into memory
const data = fs.readFileSync("large-file.csv");

// ✅ Good - streams chunk by chunk
const readStream = fs.createReadStream("large-file.csv");
const writeStream = fs.createWriteStream("output.csv");

const transform = new Transform({
  transform(chunk, encoding, callback) {
    // Process chunk
    const processed = chunk.toString().toUpperCase();
    callback(null, processed);
  },
});

readStream.pipe(transform).pipe(writeStream);
```

## 7. Load Balancing

### Nginx Configuration

```nginx
upstream nodejs {
    least_conn;  # Load balancing method
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
    server 127.0.0.1:3004;
}

server {
    listen 80;

    location / {
        proxy_pass http://nodejs;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

## 8. Monitoring & Profiling

```javascript
// Built-in profiler
node --prof app.js
node --prof-process isolate-*.log > processed.txt

// Clinic.js
npm install clinic -g
clinic doctor -- node app.js
clinic flame -- node app.js
clinic bubbleprof -- node app.js
```

## 9. Performance Checklist

- [ ] Use clustering/PM2 for multi-core
- [ ] Implement caching (Redis)
- [ ] Optimize database queries
- [ ] Use connection pooling
- [ ] Stream large files
- [ ] Compress responses (gzip)
- [ ] Use CDN for static assets
- [ ] Monitor memory usage
- [ ] Set up health checks
- [ ] Use async operations

# Performance & Scaling trong Node.js

## 1. Khái niệm

### Tại sao cần quan tâm Performance?

- **User Experience:** Response nhanh = user hài lòng
- **Cost:** Tối ưu = ít server = tiết kiệm tiền
- **Scalability:** Handle được nhiều users hơn

### Scaling là gì?

| Loại                   | Mô tả                      | Ví dụ                |
| ---------------------- | -------------------------- | -------------------- |
| **Vertical Scaling**   | Nâng cấp server (CPU, RAM) | 2 CPU → 8 CPU        |
| **Horizontal Scaling** | Thêm nhiều servers         | 1 server → 4 servers |

## 2. Clustering

### Khái niệm

**Clustering** cho phép tận dụng multi-core CPU bằng cách chạy nhiều Node.js processes.

Node.js mặc định chạy single-threaded, chỉ dùng 1 CPU core. Clustering tạo nhiều worker processes, mỗi worker dùng 1 core.

### Implementation

```javascript
const cluster = require("cluster");
const http = require("http");
const numCPUs = require("os").cpus().length;

if (cluster.isPrimary) {
  console.log(`Primary ${process.pid} is running`);
  console.log(`Forking ${numCPUs} workers...`);

  // Fork workers (1 per CPU core)
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  // Restart worker nếu crash
  cluster.on("exit", (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    console.log("Starting a new worker...");
    cluster.fork();
  });
} else {
  // Workers share TCP connection
  http
    .createServer((req, res) => {
      res.writeHead(200);
      res.end(`Hello from worker ${process.pid}\n`);
    })
    .listen(8000);

  console.log(`Worker ${process.pid} started`);
}
```

### PM2 - Production Process Manager

```bash
npm install pm2 -g

# Start với cluster mode
pm2 start app.js -i max  # max = số CPU cores
pm2 start app.js -i 4    # 4 instances

# Commands
pm2 list              # Xem processes
pm2 logs              # Xem logs
pm2 monit             # Monitor realtime
pm2 reload app        # Zero-downtime reload
pm2 stop app
pm2 delete app
```

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: "my-app",
      script: "./app.js",
      instances: "max", // Cluster mode
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

// pm2 start ecosystem.config.js --env production
```

## 3. Worker Threads

### Khái niệm

**Worker Threads** cho phép chạy JavaScript trong threads riêng biệt, phù hợp cho CPU-intensive tasks mà không block Event Loop.

### Khi nào dùng?

| Clustering         | Worker Threads                |
| ------------------ | ----------------------------- |
| Scale HTTP servers | CPU-intensive tasks           |
| Nhiều processes    | Nhiều threads trong 1 process |
| Không share memory | Có thể share memory           |

### Implementation

```javascript
// main.js
const { Worker, isMainThread, parentPort, workerData } = require("worker_threads");

if (isMainThread) {
  // Main thread
  console.log("Main thread starting...");

  const worker = new Worker(__filename, {
    workerData: { num: 40 },
  });

  worker.on("message", (result) => {
    console.log("Fibonacci result:", result);
  });

  worker.on("error", (err) => {
    console.error("Worker error:", err);
  });

  worker.on("exit", (code) => {
    console.log("Worker exited with code:", code);
  });

  // Main thread vẫn responsive
  console.log("Main thread still running...");
} else {
  // Worker thread
  const { num } = workerData;

  // CPU-intensive task
  function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
  }

  const result = fibonacci(num);
  parentPort.postMessage(result);
}
```

### Worker Pool

```javascript
const { StaticPool } = require("node-worker-threads-pool");

// Tạo pool với 4 workers
const pool = new StaticPool({
  size: 4,
  task: "./heavy-task.js",
});

// Sử dụng
async function processData(data) {
  const result = await pool.exec(data);
  return result;
}
```

## 4. Caching

### Khái niệm

**Caching** lưu trữ kết quả của operations tốn kém để tái sử dụng, giảm load cho database và tăng tốc response.

### In-Memory Cache

```javascript
const NodeCache = require("node-cache");

const cache = new NodeCache({
  stdTTL: 600, // 10 phút
  checkperiod: 120, // Check expired mỗi 2 phút
});

async function getUser(id) {
  const cacheKey = `user:${id}`;

  // Check cache
  const cached = cache.get(cacheKey);
  if (cached) {
    console.log("Cache hit!");
    return cached;
  }

  // Cache miss - fetch from DB
  console.log("Cache miss, fetching from DB...");
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
  if (cached) {
    return JSON.parse(cached);
  }

  // Fetch from DB
  const user = await User.findById(id);

  // Store in cache với TTL
  await redis.setex(cacheKey, 600, JSON.stringify(user));

  return user;
}

// Cache invalidation
async function updateUser(id, data) {
  await User.findByIdAndUpdate(id, data);
  await redis.del(`user:${id}`); // Xóa cache
}
```

### Cache Patterns

```javascript
// Cache-Aside (Lazy Loading)
// Đọc: Check cache → Miss → Fetch DB → Store cache
// Ghi: Update DB → Delete cache

// Write-Through
// Ghi: Update DB → Update cache (đồng thời)

// Write-Behind
// Ghi: Update cache → Queue → Update DB (async)
```

## 5. Database Optimization

### Connection Pooling

```javascript
// PostgreSQL
const { Pool } = require("pg");

const pool = new Pool({
  max: 20, // Max connections
  idleTimeoutMillis: 30000, // Close idle sau 30s
  connectionTimeoutMillis: 2000,
});

// MongoDB
mongoose.connect(uri, {
  maxPoolSize: 10,
  minPoolSize: 5,
  serverSelectionTimeoutMS: 5000,
});
```

### N+1 Query Problem

```javascript
// ❌ N+1 Problem: 1 + N queries
const users = await User.find(); // 1 query
for (const user of users) {
  const posts = await Post.find({ authorId: user.id }); // N queries!
}

// ✅ Eager Loading: 2 queries
const users = await User.find().populate("posts");

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

## 6. Memory Management

### Detect Memory Leaks

```javascript
// Check memory usage
const used = process.memoryUsage();
console.log({
  rss: `${Math.round(used.rss / 1024 / 1024)} MB`, // Total memory
  heapTotal: `${Math.round(used.heapTotal / 1024 / 1024)} MB`,
  heapUsed: `${Math.round(used.heapUsed / 1024 / 1024)} MB`,
  external: `${Math.round(used.external / 1024 / 1024)} MB`,
});

// Heap snapshot
const v8 = require("v8");
v8.writeHeapSnapshot(); // Tạo file .heapsnapshot
```

### Common Memory Leaks

```javascript
// ❌ Global variables tích tụ
global.cache = [];
global.cache.push(data); // Grows forever

// ❌ Event listeners không remove
emitter.on("event", handler);
// Fix: emitter.off('event', handler);

// ❌ Closures giữ references
function createLeak() {
  const bigData = new Array(1000000);
  return () => bigData.length; // bigData không được GC
}

// ❌ Timers không clear
const interval = setInterval(() => {}, 1000);
// Fix: clearInterval(interval);
```

## 7. Streams cho Large Data

```javascript
const fs = require("fs");

// ❌ Bad: Load toàn bộ file vào memory
const data = fs.readFileSync("large-file.csv"); // 2GB in memory!

// ✅ Good: Stream từng chunk
const readStream = fs.createReadStream("large-file.csv");
const writeStream = fs.createWriteStream("output.csv");

readStream.pipe(transformStream).pipe(writeStream);

// Pipeline với error handling
const { pipeline } = require("stream/promises");

await pipeline(fs.createReadStream("input.txt"), zlib.createGzip(), fs.createWriteStream("input.txt.gz"));
```

## 8. Load Balancing với Nginx

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

## 9. Performance Checklist

- [ ] Clustering/PM2 cho multi-core
- [ ] Caching (Redis)
- [ ] Database indexing
- [ ] Connection pooling
- [ ] Gzip compression
- [ ] CDN cho static assets
- [ ] Avoid N+1 queries
- [ ] Stream large files
- [ ] Monitor memory usage
- [ ] Load balancing

## 10. Tổng kết

**Clustering:** Tận dụng multi-core CPU với PM2.

**Worker Threads:** CPU-intensive tasks không block Event Loop.

**Caching:** Redis để giảm database load.

**Database:** Indexing, connection pooling, avoid N+1.

**Memory:** Monitor và tránh memory leaks.

**Streams:** Xử lý large files hiệu quả.

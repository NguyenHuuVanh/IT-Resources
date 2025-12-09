# DevOps & Deployment cho Node.js

## 1. Khái niệm

### DevOps là gì?

**DevOps** là văn hóa và practices kết hợp Development và Operations, tự động hóa quy trình từ code đến production.

### CI/CD là gì?

| Khái niệm                       | Mô tả                               |
| ------------------------------- | ----------------------------------- |
| **CI (Continuous Integration)** | Tự động build và test khi push code |
| **CD (Continuous Delivery)**    | Tự động deploy đến staging          |
| **CD (Continuous Deployment)**  | Tự động deploy đến production       |

## 2. Docker

### Khái niệm

**Docker** là platform để đóng gói ứng dụng và dependencies vào containers, đảm bảo chạy giống nhau ở mọi môi trường.

### Dockerfile

```dockerfile
# Build stage
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Production stage
FROM node:20-alpine

WORKDIR /app

# Create non-root user (security)
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy from builder
COPY --from=builder /app/node_modules ./node_modules
COPY . .

# Change ownership
RUN chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# Start command
CMD ["node", "src/index.js"]
```

### Docker Compose

```yaml
# docker-compose.yml
version: "3.8"

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    restart: unless-stopped
    networks:
      - app-network

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  postgres_data:
  redis_data:
```

### Docker Commands

```bash
# Build image
docker build -t myapp .

# Run container
docker run -p 3000:3000 -e NODE_ENV=production myapp

# Docker Compose
docker-compose up -d        # Start
docker-compose logs -f app  # Logs
docker-compose down         # Stop
docker-compose ps           # List
```

## 3. Environment Variables

### Khái niệm

**Environment Variables** lưu trữ configuration bên ngoài code, cho phép thay đổi behavior mà không cần rebuild.

### .env file

```bash
# .env
NODE_ENV=development
PORT=3000

# Database
DATABASE_URL=postgres://localhost:5432/mydb

# JWT
JWT_SECRET=your-super-secret-key-at-least-32-chars
JWT_EXPIRES_IN=15m

# Redis
REDIS_URL=redis://localhost:6379

# External APIs
STRIPE_KEY=sk_test_xxx
SENDGRID_API_KEY=SG.xxx
```

### Config Module

```javascript
// config/index.js
require("dotenv").config();
const Joi = require("joi");

// Validation schema
const envSchema = Joi.object({
  NODE_ENV: Joi.string().valid("development", "production", "test").required(),
  PORT: Joi.number().default(3000),
  DATABASE_URL: Joi.string().required(),
  JWT_SECRET: Joi.string().min(32).required(),
  JWT_EXPIRES_IN: Joi.string().default("15m"),
  REDIS_URL: Joi.string().required(),
}).unknown(); // Cho phép env vars khác

// Validate
const { error, value: envVars } = envSchema.validate(process.env);
if (error) {
  throw new Error(`Config validation error: ${error.message}`);
}

// Export config
module.exports = {
  env: envVars.NODE_ENV,
  port: envVars.PORT,
  db: {
    url: envVars.DATABASE_URL,
  },
  jwt: {
    secret: envVars.JWT_SECRET,
    expiresIn: envVars.JWT_EXPIRES_IN,
  },
  redis: {
    url: envVars.REDIS_URL,
  },
};
```

## 4. CI/CD với GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test
        env:
          DATABASE_URL: postgres://postgres:postgres@localhost:5432/test
          JWT_SECRET: test-secret-key-for-ci-testing-only

      - name: Build
        run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to production
        run: |
          echo "Deploying to production..."
          # Add deployment commands here
```

## 5. Logging

### Winston

```javascript
const winston = require("winston");

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || "info",
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: "my-app" },
  transports: [
    // Error logs
    new winston.transports.File({
      filename: "logs/error.log",
      level: "error",
    }),
    // All logs
    new winston.transports.File({
      filename: "logs/combined.log",
    }),
  ],
});

// Console logging in development
if (process.env.NODE_ENV !== "production") {
  logger.add(
    new winston.transports.Console({
      format: winston.format.combine(winston.format.colorize(), winston.format.simple()),
    })
  );
}

// Usage
logger.info("Server started", { port: 3000 });
logger.error("Database error", { error: err.message, stack: err.stack });
logger.warn("Deprecated API called", { endpoint: "/old-api" });

module.exports = logger;
```

## 6. Health Checks

```javascript
// Health check endpoints
app.get("/health", async (req, res) => {
  const health = {
    status: "ok",
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    checks: {},
  };

  // Database check
  try {
    await db.query("SELECT 1");
    health.checks.database = "ok";
  } catch (error) {
    health.checks.database = "error";
    health.status = "degraded";
  }

  // Redis check
  try {
    await redis.ping();
    health.checks.redis = "ok";
  } catch (error) {
    health.checks.redis = "error";
    health.status = "degraded";
  }

  // Memory check
  const memUsage = process.memoryUsage();
  health.checks.memory = {
    heapUsed: `${Math.round(memUsage.heapUsed / 1024 / 1024)} MB`,
    heapTotal: `${Math.round(memUsage.heapTotal / 1024 / 1024)} MB`,
  };

  const statusCode = health.status === "ok" ? 200 : 503;
  res.status(statusCode).json(health);
});

// Kubernetes probes
app.get("/ready", (req, res) => {
  // Readiness: App sẵn sàng nhận traffic?
  res.status(200).send("Ready");
});

app.get("/live", (req, res) => {
  // Liveness: App còn sống?
  res.status(200).send("Alive");
});
```

## 7. Graceful Shutdown

```javascript
const server = app.listen(PORT);

async function gracefulShutdown(signal) {
  console.log(`${signal} received. Starting graceful shutdown...`);

  // Stop accepting new connections
  server.close(async () => {
    console.log("HTTP server closed");

    try {
      // Close database connections
      await db.end();
      console.log("Database connections closed");

      // Close Redis
      await redis.quit();
      console.log("Redis connection closed");

      // Close other resources...

      console.log("Graceful shutdown complete");
      process.exit(0);
    } catch (error) {
      console.error("Error during shutdown:", error);
      process.exit(1);
    }
  });

  // Force shutdown after timeout
  setTimeout(() => {
    console.error("Forced shutdown after timeout");
    process.exit(1);
  }, 30000); // 30 seconds
}

// Handle signals
process.on("SIGTERM", () => gracefulShutdown("SIGTERM"));
process.on("SIGINT", () => gracefulShutdown("SIGINT"));

// Handle uncaught errors
process.on("uncaughtException", (error) => {
  console.error("Uncaught Exception:", error);
  gracefulShutdown("uncaughtException");
});

process.on("unhandledRejection", (reason, promise) => {
  console.error("Unhandled Rejection at:", promise, "reason:", reason);
});
```

## 8. Nginx Reverse Proxy

```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;

    # SSL
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # Gzip
    gzip on;
    gzip_types text/plain application/json application/javascript text/css;

    # Static files
    location /static {
        alias /var/www/static;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # API proxy
    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 9. Deployment Checklist

- [ ] Environment variables configured
- [ ] Production dependencies only (`npm ci --only=production`)
- [ ] Logging configured
- [ ] Health checks implemented
- [ ] Graceful shutdown handled
- [ ] SSL/TLS enabled
- [ ] Gzip compression
- [ ] Security headers (Helmet)
- [ ] Rate limiting
- [ ] Error monitoring (Sentry)
- [ ] PM2 or Docker for process management
- [ ] Database migrations run
- [ ] Backup strategy

## 10. Tổng kết

**Docker:** Đóng gói app và dependencies vào containers.

**Environment Variables:** Config bên ngoài code.

**CI/CD:** Tự động test và deploy.

**Logging:** Winston cho structured logging.

**Health Checks:** Monitor app health.

**Graceful Shutdown:** Đóng connections đúng cách.

**Nginx:** Reverse proxy, SSL, static files.

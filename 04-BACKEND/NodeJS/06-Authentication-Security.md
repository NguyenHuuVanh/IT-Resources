# Authentication & Security trong Node.js

## 1. JWT (JSON Web Token)

### Cấu trúc JWT

```
Header.Payload.Signature

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4ifQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

- **Header**: Algorithm & token type
- **Payload**: Claims (data)
- **Signature**: Verify token integrity

### Implementation

```javascript
const jwt = require("jsonwebtoken");

const SECRET = process.env.JWT_SECRET;
const REFRESH_SECRET = process.env.JWT_REFRESH_SECRET;

// Generate tokens
function generateTokens(user) {
  const accessToken = jwt.sign({ userId: user.id, email: user.email }, SECRET, { expiresIn: "15m" });

  const refreshToken = jwt.sign({ userId: user.id }, REFRESH_SECRET, { expiresIn: "7d" });

  return { accessToken, refreshToken };
}

// Verify token
function verifyToken(token) {
  try {
    return jwt.verify(token, SECRET);
  } catch (error) {
    return null;
  }
}
```

// Auth middleware
function authMiddleware(req, res, next) {
const authHeader = req.headers.authorization;

if (!authHeader || !authHeader.startsWith('Bearer ')) {
return res.status(401).json({ error: 'No token provided' });
}

const token = authHeader.split(' ')[1];
const decoded = verifyToken(token);

if (!decoded) {
return res.status(401).json({ error: 'Invalid token' });
}

req.user = decoded;
next();
}

// Refresh token endpoint
app.post('/refresh', async (req, res) => {
const { refreshToken } = req.body;

try {
const decoded = jwt.verify(refreshToken, REFRESH_SECRET);
const user = await User.findById(decoded.userId);

    if (!user) {
      return res.status(401).json({ error: 'User not found' });
    }

    const tokens = generateTokens(user);
    res.json(tokens);

} catch (error) {
res.status(401).json({ error: 'Invalid refresh token' });
}
});

````

### Token Rotation Strategy
```javascript
// Lưu refresh token vào database
const refreshTokenSchema = new Schema({
  token: String,
  userId: ObjectId,
  expiresAt: Date,
  isRevoked: { type: Boolean, default: false }
});

// Khi refresh: tạo token mới, revoke token cũ
async function rotateRefreshToken(oldToken) {
  const tokenDoc = await RefreshToken.findOne({ token: oldToken });

  if (!tokenDoc || tokenDoc.isRevoked) {
    // Possible token reuse attack - revoke all user tokens
    await RefreshToken.updateMany(
      { userId: tokenDoc.userId },
      { isRevoked: true }
    );
    throw new Error('Token reuse detected');
  }

  // Revoke old token
  tokenDoc.isRevoked = true;
  await tokenDoc.save();

  // Generate new tokens
  return generateTokens(user);
}
````

## 2. Session-based Authentication

```javascript
const session = require("express-session");
const RedisStore = require("connect-redis").default;
const redis = require("redis");

const redisClient = redis.createClient();

app.use(
  session({
    store: new RedisStore({ client: redisClient }),
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
      secure: process.env.NODE_ENV === "production", // HTTPS only
      httpOnly: true, // Prevent XSS
      maxAge: 24 * 60 * 60 * 1000, // 24 hours
      sameSite: "strict", // CSRF protection
    },
  })
);

// Login
app.post("/login", async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });

  if (!user || !(await bcrypt.compare(password, user.password))) {
    return res.status(401).json({ error: "Invalid credentials" });
  }

  req.session.userId = user.id;
  res.json({ message: "Logged in" });
});

// Logout
app.post("/logout", (req, res) => {
  req.session.destroy();
  res.json({ message: "Logged out" });
});
```

### JWT vs Session

| Feature     | JWT                          | Session                  |
| ----------- | ---------------------------- | ------------------------ |
| Storage     | Client (localStorage/cookie) | Server (memory/Redis)    |
| Scalability | Stateless, dễ scale          | Cần shared session store |
| Revocation  | Khó (cần blacklist)          | Dễ (xóa session)         |
| Size        | Lớn hơn                      | Chỉ session ID           |
| Mobile      | Phù hợp                      | Ít phù hợp               |

## 3. Password Hashing

```javascript
const bcrypt = require("bcrypt");
const argon2 = require("argon2");

// Bcrypt
const SALT_ROUNDS = 12;

async function hashPassword(password) {
  return bcrypt.hash(password, SALT_ROUNDS);
}

async function verifyPassword(password, hash) {
  return bcrypt.compare(password, hash);
}

// Argon2 (recommended - winner of Password Hashing Competition)
async function hashPasswordArgon2(password) {
  return argon2.hash(password, {
    type: argon2.argon2id,
    memoryCost: 65536,
    timeCost: 3,
    parallelism: 4,
  });
}

async function verifyPasswordArgon2(password, hash) {
  return argon2.verify(hash, password);
}
```

## 4. OAuth 2.0 / OpenID Connect

```javascript
const passport = require("passport");
const GoogleStrategy = require("passport-google-oauth20").Strategy;

passport.use(
  new GoogleStrategy(
    {
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL: "/auth/google/callback",
    },
    async (accessToken, refreshToken, profile, done) => {
      try {
        let user = await User.findOne({ googleId: profile.id });

        if (!user) {
          user = await User.create({
            googleId: profile.id,
            email: profile.emails[0].value,
            name: profile.displayName,
          });
        }

        done(null, user);
      } catch (error) {
        done(error, null);
      }
    }
  )
);

// Routes
app.get("/auth/google", passport.authenticate("google", { scope: ["profile", "email"] }));

app.get("/auth/google/callback", passport.authenticate("google", { failureRedirect: "/login" }), (req, res) => {
  const tokens = generateTokens(req.user);
  res.redirect(`/dashboard?token=${tokens.accessToken}`);
});
```

## 5. Security Best Practices

### Helmet.js - Security Headers

```javascript
const helmet = require("helmet");

app.use(helmet()); // Adds various security headers

// Hoặc configure riêng
app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
    },
  })
);
app.use(helmet.xssFilter());
app.use(helmet.noSniff());
app.use(helmet.frameguard({ action: "deny" }));
```

### CORS Configuration

```javascript
const cors = require("cors");

app.use(
  cors({
    origin: ["https://myapp.com", "https://admin.myapp.com"],
    methods: ["GET", "POST", "PUT", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"],
    credentials: true,
    maxAge: 86400, // 24 hours
  })
);
```

### Rate Limiting

```javascript
const rateLimit = require("express-rate-limit");

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: "Too many requests, please try again later",
  standardHeaders: true,
  legacyHeaders: false,
});

app.use("/api/", limiter);

// Stricter limit for auth endpoints
const authLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5, // 5 attempts
  message: "Too many login attempts",
});

app.use("/api/login", authLimiter);
```

### Input Validation

```javascript
const Joi = require("joi");
const { body, validationResult } = require("express-validator");

// Joi
const userSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string()
    .min(8)
    .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
    .required(),
  name: Joi.string().max(100),
});

app.post("/register", async (req, res) => {
  const { error, value } = userSchema.validate(req.body);
  if (error) {
    return res.status(400).json({ error: error.details[0].message });
  }
  // Process validated data
});

// express-validator
app.post(
  "/register",
  body("email").isEmail().normalizeEmail(),
  body("password")
    .isLength({ min: 8 })
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/),
  body("name").trim().escape(),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    // Process validated data
  }
);
```

### SQL Injection Prevention

```javascript
// ❌ Bad - SQL Injection vulnerable
const query = `SELECT * FROM users WHERE email = '${email}'`;

// ✅ Good - Parameterized query
const query = "SELECT * FROM users WHERE email = $1";
const result = await pool.query(query, [email]);

// ✅ Good - ORM (auto-escaped)
const user = await User.findOne({ where: { email } });
```

### XSS Prevention

```javascript
const xss = require("xss");

// Sanitize user input
const sanitizedInput = xss(userInput);

// In templates - use auto-escaping
// EJS: <%= userInput %> (escaped)
// EJS: <%- userInput %> (unescaped - avoid!)
```

## 6. CSRF Protection

```javascript
const csrf = require("csurf");

const csrfProtection = csrf({ cookie: true });

app.get("/form", csrfProtection, (req, res) => {
  res.render("form", { csrfToken: req.csrfToken() });
});

app.post("/submit", csrfProtection, (req, res) => {
  // CSRF token validated automatically
  res.send("Data submitted");
});
```

## 7. Security Checklist

- [ ] HTTPS everywhere
- [ ] Strong password hashing (bcrypt/argon2)
- [ ] JWT với expiration ngắn + refresh token
- [ ] Rate limiting
- [ ] Input validation & sanitization
- [ ] Parameterized queries
- [ ] Security headers (Helmet)
- [ ] CORS configuration
- [ ] CSRF protection
- [ ] Secure cookie settings
- [ ] Environment variables cho secrets
- [ ] Logging & monitoring
- [ ] Regular dependency updates

# Testing trong Node.js

## 1. Khái niệm

### Tại sao cần Testing?

- **Phát hiện bugs sớm:** Trước khi deploy
- **Tự tin refactor:** Biết code vẫn hoạt động
- **Documentation:** Tests mô tả cách code hoạt động
- **Regression prevention:** Tránh lỗi cũ quay lại

### Các loại Testing

| Loại                 | Mô tả                              | Tốc độ     | Coverage   |
| -------------------- | ---------------------------------- | ---------- | ---------- |
| **Unit Test**        | Test từng function/module riêng lẻ | Nhanh      | Hẹp        |
| **Integration Test** | Test nhiều components kết hợp      | Trung bình | Trung bình |
| **E2E Test**         | Test toàn bộ flow từ đầu đến cuối  | Chậm       | Rộng       |

### Testing Pyramid

```
        /\
       /  \      E2E Tests (ít)
      /----\
     /      \    Integration Tests (vừa)
    /--------\
   /          \  Unit Tests (nhiều)
  /------------\
```

## 2. Jest - Testing Framework

### Khái niệm

**Jest** là testing framework phổ biến nhất cho JavaScript, cung cấp test runner, assertions, mocking trong một package.

### Cài đặt

```bash
npm install jest @types/jest -D
```

```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

### Cấu trúc Test cơ bản

```javascript
// math.js
function add(a, b) {
  return a + b;
}

function divide(a, b) {
  if (b === 0) throw new Error("Cannot divide by zero");
  return a / b;
}

module.exports = { add, divide };

// math.test.js
const { add, divide } = require("./math");

describe("Math functions", () => {
  describe("add", () => {
    test("adds 1 + 2 to equal 3", () => {
      expect(add(1, 2)).toBe(3);
    });

    test("adds negative numbers", () => {
      expect(add(-1, -2)).toBe(-3);
    });

    test("adds zero", () => {
      expect(add(5, 0)).toBe(5);
    });
  });

  describe("divide", () => {
    test("divides 10 / 2 to equal 5", () => {
      expect(divide(10, 2)).toBe(5);
    });

    test("throws error when dividing by zero", () => {
      expect(() => divide(10, 0)).toThrow("Cannot divide by zero");
    });
  });
});
```

### Jest Matchers

```javascript
// Equality
expect(value).toBe(3); // === (primitive)
expect(value).toEqual({ a: 1 }); // Deep equality (objects)
expect(value).toStrictEqual(obj); // Strict deep equality

// Truthiness
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeUndefined();
expect(value).toBeDefined();

// Numbers
expect(value).toBeGreaterThan(3);
expect(value).toBeGreaterThanOrEqual(3);
expect(value).toBeLessThan(5);
expect(value).toBeCloseTo(0.3, 5); // Floating point

// Strings
expect(string).toMatch(/regex/);
expect(string).toContain("substring");

// Arrays
expect(array).toContain(item);
expect(array).toHaveLength(3);
expect(array).toContainEqual({ name: "John" });

// Objects
expect(obj).toHaveProperty("key");
expect(obj).toHaveProperty("key", "value");
expect(obj).toMatchObject({ key: "value" });

// Exceptions
expect(() => fn()).toThrow();
expect(() => fn()).toThrow(Error);
expect(() => fn()).toThrow("error message");

// Async
await expect(asyncFn()).resolves.toBe(value);
await expect(asyncFn()).rejects.toThrow();

// Negation
expect(value).not.toBe(3);
```

## 3. Mocking

### Khái niệm

**Mocking** là kỹ thuật thay thế dependencies thật bằng fake implementations để isolate unit đang test.

### Mock Functions

```javascript
// Tạo mock function
const mockFn = jest.fn();

// Mock return value
mockFn.mockReturnValue(42);
mockFn.mockReturnValueOnce(10); // Chỉ lần đầu

// Mock async
mockFn.mockResolvedValue({ data: "async" });
mockFn.mockRejectedValue(new Error("Failed"));

// Mock implementation
mockFn.mockImplementation((a, b) => a + b);

// Assertions
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledTimes(2);
expect(mockFn).toHaveBeenCalledWith(arg1, arg2);
expect(mockFn).toHaveBeenLastCalledWith(arg);
```

### Mock Modules

```javascript
// __mocks__/axios.js
module.exports = {
  get: jest.fn(() => Promise.resolve({ data: {} })),
  post: jest.fn(() => Promise.resolve({ data: {} })),
};

// userService.test.js
jest.mock("axios");
const axios = require("axios");
const { getUsers } = require("./userService");

test("fetches users", async () => {
  // Arrange
  const users = [{ id: 1, name: "John" }];
  axios.get.mockResolvedValue({ data: users });

  // Act
  const result = await getUsers();

  // Assert
  expect(axios.get).toHaveBeenCalledWith("/api/users");
  expect(result).toEqual(users);
});
```

### Spy on Methods

```javascript
const obj = {
  method: () => "original",
};

const spy = jest.spyOn(obj, "method");
spy.mockReturnValue("mocked");

obj.method(); // 'mocked'

expect(spy).toHaveBeenCalled();

spy.mockRestore(); // Restore original
```

## 4. Testing Async Code

```javascript
// Callbacks
test("callback test", (done) => {
  fetchData((data) => {
    expect(data).toBe("data");
    done(); // Báo test hoàn thành
  });
});

// Promises
test("promise test", () => {
  return fetchData().then((data) => {
    expect(data).toBe("data");
  });
});

// Async/Await
test("async test", async () => {
  const data = await fetchData();
  expect(data).toBe("data");
});

// Resolves/Rejects
test("resolves", async () => {
  await expect(fetchData()).resolves.toBe("data");
});

test("rejects", async () => {
  await expect(fetchBadData()).rejects.toThrow("Error");
});
```

## 5. Integration Testing với Supertest

### Khái niệm

**Supertest** cho phép test HTTP endpoints mà không cần start server thật.

```bash
npm install supertest -D
```

```javascript
// app.js
const express = require("express");
const app = express();

app.use(express.json());

app.get("/api/users", (req, res) => {
  res.json({ users: [] });
});

app.post("/api/users", (req, res) => {
  const { name, email } = req.body;
  res.status(201).json({ id: 1, name, email });
});

module.exports = app;

// app.test.js
const request = require("supertest");
const app = require("./app");

describe("User API", () => {
  describe("GET /api/users", () => {
    test("should return all users", async () => {
      const res = await request(app).get("/api/users").expect("Content-Type", /json/).expect(200);

      expect(res.body).toHaveProperty("users");
      expect(Array.isArray(res.body.users)).toBe(true);
    });
  });

  describe("POST /api/users", () => {
    test("should create a new user", async () => {
      const userData = {
        name: "John Doe",
        email: "john@example.com",
      };

      const res = await request(app).post("/api/users").send(userData).expect("Content-Type", /json/).expect(201);

      expect(res.body).toHaveProperty("id");
      expect(res.body.name).toBe(userData.name);
      expect(res.body.email).toBe(userData.email);
    });

    test("should return 400 for invalid data", async () => {
      const res = await request(app)
        .post("/api/users")
        .send({ name: "" }) // Invalid
        .expect(400);

      expect(res.body).toHaveProperty("error");
    });
  });

  describe("Protected routes", () => {
    let token;

    beforeAll(async () => {
      // Login để lấy token
      const res = await request(app).post("/api/login").send({ email: "admin@test.com", password: "password" });
      token = res.body.token;
    });

    test("should access protected route with token", async () => {
      const res = await request(app).get("/api/profile").set("Authorization", `Bearer ${token}`).expect(200);

      expect(res.body).toHaveProperty("user");
    });

    test("should reject without token", async () => {
      await request(app).get("/api/profile").expect(401);
    });
  });
});
```

## 6. Database Testing

```javascript
// Dùng in-memory database cho tests
const { MongoMemoryServer } = require("mongodb-memory-server");
const mongoose = require("mongoose");
const User = require("./models/User");

let mongoServer;

beforeAll(async () => {
  // Start in-memory MongoDB
  mongoServer = await MongoMemoryServer.create();
  await mongoose.connect(mongoServer.getUri());
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});

beforeEach(async () => {
  // Clear database trước mỗi test
  await User.deleteMany({});
});

describe("User Model", () => {
  test("should create user", async () => {
    const user = await User.create({
      email: "test@example.com",
      name: "Test User",
      password: "password123",
    });

    expect(user._id).toBeDefined();
    expect(user.email).toBe("test@example.com");
  });

  test("should not create user with duplicate email", async () => {
    await User.create({
      email: "test@example.com",
      name: "User 1",
      password: "password",
    });

    await expect(
      User.create({
        email: "test@example.com",
        name: "User 2",
        password: "password",
      })
    ).rejects.toThrow();
  });
});
```

## 7. Test Coverage

```json
// package.json
{
  "jest": {
    "collectCoverageFrom": ["src/**/*.js", "!src/**/*.test.js"],
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

```bash
npm run test:coverage
```

## 8. Best Practices

### AAA Pattern

```javascript
test("should calculate total", () => {
  // Arrange - Setup
  const cart = new Cart();
  cart.addItem({ price: 100, quantity: 2 });
  cart.addItem({ price: 50, quantity: 1 });

  // Act - Execute
  const total = cart.getTotal();

  // Assert - Verify
  expect(total).toBe(250);
});
```

### Descriptive Test Names

```javascript
// ❌ Bad
test("test1", () => {});

// ✅ Good
test("should return empty array when no users exist", () => {});
test("should throw error when email is invalid", () => {});
```

### Isolate Tests

```javascript
// ❌ Bad - Tests phụ thuộc nhau
let user;
test("create user", () => {
  user = createUser();
});
test("update user", () => {
  updateUser(user.id); // Phụ thuộc test trước
});

// ✅ Good - Tests độc lập
test("create user", () => {
  const user = createUser();
  expect(user).toBeDefined();
});
test("update user", () => {
  const user = createUser(); // Tạo mới
  const updated = updateUser(user.id);
  expect(updated).toBeDefined();
});
```

## 9. Tổng kết

**Unit Test:** Test từng function riêng lẻ, nhanh, nhiều.

**Integration Test:** Test nhiều components, Supertest cho APIs.

**Mocking:** Thay thế dependencies để isolate.

**Coverage:** Đo lường % code được test.

**Best Practices:**

- AAA Pattern (Arrange, Act, Assert)
- Descriptive names
- Isolate tests
- Mock external dependencies
- Test edge cases

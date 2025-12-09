# Testing trong Node.js

## 1. Testing Frameworks

### Jest (Recommended)

```bash
npm install jest @types/jest -D
```

```javascript
// sum.js
function sum(a, b) {
  return a + b;
}
module.exports = sum;

// sum.test.js
const sum = require("./sum");

describe("sum function", () => {
  test("adds 1 + 2 to equal 3", () => {
    expect(sum(1, 2)).toBe(3);
  });

  test("adds negative numbers", () => {
    expect(sum(-1, -2)).toBe(-3);
  });
});
```

### Mocha + Chai

```bash
npm install mocha chai -D
```

```javascript
const { expect } = require("chai");
const sum = require("./sum");

describe("sum function", () => {
  it("should add 1 + 2 to equal 3", () => {
    expect(sum(1, 2)).to.equal(3);
  });
});
```

## 2. Jest Matchers

```javascript
// Equality
expect(value).toBe(3); // ===
expect(value).toEqual({ a: 1 }); // Deep equality
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
```

## 3. Mocking

### Mock Functions

```javascript
// Create mock function
const mockFn = jest.fn();
mockFn.mockReturnValue(42);
mockFn.mockReturnValueOnce(10);
mockFn.mockResolvedValue({ data: "async" });
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

// test file
jest.mock("axios");
const axios = require("axios");

test("fetches data", async () => {
  axios.get.mockResolvedValue({ data: { users: [] } });

  const result = await fetchUsers();

  expect(axios.get).toHaveBeenCalledWith("/api/users");
  expect(result).toEqual({ users: [] });
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
    done();
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
```

## 5. Integration Testing với Supertest

```bash
npm install supertest -D
```

```javascript
const request = require("supertest");
const app = require("../app");

describe("User API", () => {
  describe("GET /api/users", () => {
    it("should return all users", async () => {
      const res = await request(app).get("/api/users").expect("Content-Type", /json/).expect(200);

      expect(res.body).toHaveProperty("users");
      expect(Array.isArray(res.body.users)).toBe(true);
    });
  });

  describe("POST /api/users", () => {
    it("should create a new user", async () => {
      const userData = {
        email: "test@example.com",
        name: "Test User",
      };

      const res = await request(app).post("/api/users").send(userData).expect(201);

      expect(res.body).toHaveProperty("id");
      expect(res.body.email).toBe(userData.email);
    });

    it("should return 400 for invalid data", async () => {
      const res = await request(app).post("/api/users").send({ name: "No Email" }).expect(400);

      expect(res.body).toHaveProperty("error");
    });
  });

  describe("Protected routes", () => {
    let token;

    beforeAll(async () => {
      const res = await request(app).post("/api/login").send({ email: "admin@test.com", password: "password" });
      token = res.body.token;
    });

    it("should access protected route with token", async () => {
      const res = await request(app).get("/api/profile").set("Authorization", `Bearer ${token}`).expect(200);

      expect(res.body).toHaveProperty("user");
    });
  });
});
```

## 6. Database Testing

```javascript
const { MongoMemoryServer } = require("mongodb-memory-server");
const mongoose = require("mongoose");

let mongoServer;

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  await mongoose.connect(mongoServer.getUri());
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});

beforeEach(async () => {
  // Clear all collections
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});

test("should create user", async () => {
  const user = await User.create({
    email: "test@example.com",
    name: "Test",
  });

  expect(user._id).toBeDefined();
  expect(user.email).toBe("test@example.com");
});
```

## 7. Test Coverage

```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:coverage": "jest --coverage"
  },
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

## 8. Best Practices

1. **AAA Pattern**: Arrange, Act, Assert
2. **One assertion per test** (khi có thể)
3. **Descriptive test names**
4. **Test behavior, not implementation**
5. **Isolate tests** - không phụ thuộc lẫn nhau
6. **Mock external dependencies**
7. **Use factories** cho test data
8. **Clean up** sau mỗi test

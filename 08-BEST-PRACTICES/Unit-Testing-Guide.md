# Unit Testing - Hướng dẫn toàn diện (Jest & React Testing Library)

## 📌 Mục lục

1. [Tổng quan về Testing](#1-tổng-quan-về-testing)
2. [Jest Fundamentals](#2-jest-fundamentals)
3. [Jest Matchers](#3-jest-matchers)
4. [Mocking trong Jest](#4-mocking-trong-jest)
5. [Testing Async Code](#5-testing-async-code)
6. [React Testing Library](#6-react-testing-library)
7. [Testing React Components](#7-testing-react-components)
8. [Testing Hooks](#8-testing-hooks)
9. [Testing Forms & User Events](#9-testing-forms--user-events)
10. [Testing API Calls](#10-testing-api-calls)
11. [Best Practices](#11-best-practices)
12. [Code Coverage](#12-code-coverage)

---

## 1. Tổng quan về Testing

### 🎯 Tại sao cần Testing?

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      LỢI ÍCH CỦA TESTING                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ✓ Phát hiện bugs sớm         Tiết kiệm thời gian và chi phí           │
│  ✓ Tự tin refactor            Code có tests = refactor an toàn         │
│  ✓ Documentation              Tests mô tả cách code hoạt động          │
│  ✓ Better design              Viết tests → code modular hơn            │
│  ✓ Regression prevention      Tránh bugs quay lại                      │
│  ✓ CI/CD integration          Tự động kiểm tra trước deploy            │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📊 Testing Pyramid

```
                    ┌───────────┐
                    │    E2E    │  ← Ít tests, chậm, đắt
                    │  (Cypress)│     Test toàn bộ flow
                    ├───────────┤
                    │           │
                    │Integration│  ← Vừa phải
                    │   Tests   │     Test nhiều units cùng nhau
                    │           │
                    ├───────────┤
                    │           │
                    │           │
                    │   Unit    │  ← Nhiều tests, nhanh, rẻ
                    │   Tests   │     Test từng function/component
                    │           │
                    │           │
                    └───────────┘
```

### 📋 Các loại Testing

| Loại                 | Mô tả                                | Tools               |
| -------------------- | ------------------------------------ | ------------------- |
| **Unit Test**        | Test từng function/component độc lập | Jest, Vitest        |
| **Integration Test** | Test nhiều units làm việc cùng nhau  | Jest, RTL           |
| **E2E Test**         | Test toàn bộ user flow               | Cypress, Playwright |
| **Snapshot Test**    | So sánh output với snapshot đã lưu   | Jest                |
| **Visual Test**      | So sánh UI screenshots               | Chromatic, Percy    |

---

## 2. Jest Fundamentals

### 🔧 Cài đặt

```bash
# Cài đặt Jest
npm install --save-dev jest

# Với TypeScript
npm install --save-dev jest @types/jest ts-jest

# Với React
npm install --save-dev jest @testing-library/react @testing-library/jest-dom
```

### 📁 Cấu trúc file test

```
src/
├── utils/
│   ├── math.js
│   └── math.test.js        # Cùng thư mục
├── components/
│   ├── Button.jsx
│   └── Button.test.jsx
└── __tests__/              # Hoặc thư mục riêng
    └── integration.test.js
```

### 💻 Cú pháp cơ bản

```javascript
// math.js
export function add(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}

export function divide(a, b) {
  if (b === 0) throw new Error("Cannot divide by zero");
  return a / b;
}
```

```javascript
// math.test.js
import { add, multiply, divide } from "./math";

// describe: Nhóm các tests liên quan
describe("Math utilities", () => {
  // test hoặc it: Định nghĩa một test case
  test("add should return sum of two numbers", () => {
    expect(add(2, 3)).toBe(5);
    expect(add(-1, 1)).toBe(0);
    expect(add(0, 0)).toBe(0);
  });

  it("multiply should return product of two numbers", () => {
    expect(multiply(2, 3)).toBe(6);
    expect(multiply(-2, 3)).toBe(-6);
    expect(multiply(0, 100)).toBe(0);
  });

  // Nested describe
  describe("divide", () => {
    it("should return quotient of two numbers", () => {
      expect(divide(6, 2)).toBe(3);
      expect(divide(10, 4)).toBe(2.5);
    });

    it("should throw error when dividing by zero", () => {
      expect(() => divide(5, 0)).toThrow("Cannot divide by zero");
    });
  });
});
```

### 🔄 Setup và Teardown

```javascript
describe("Database tests", () => {
  let db;

  // Chạy MỘT LẦN trước TẤT CẢ tests trong describe
  beforeAll(async () => {
    db = await connectDatabase();
    console.log("Connected to database");
  });

  // Chạy MỘT LẦN sau TẤT CẢ tests trong describe
  afterAll(async () => {
    await db.disconnect();
    console.log("Disconnected from database");
  });

  // Chạy TRƯỚC MỖI test
  beforeEach(async () => {
    await db.clear();
    console.log("Cleared database");
  });

  // Chạy SAU MỖI test
  afterEach(() => {
    console.log("Test completed");
  });

  test("should insert user", async () => {
    await db.insert({ name: "John" });
    const users = await db.findAll();
    expect(users).toHaveLength(1);
  });

  test("should find user by name", async () => {
    await db.insert({ name: "Jane" });
    const user = await db.findByName("Jane");
    expect(user.name).toBe("Jane");
  });
});
```

### 🏃 Chạy Tests

```bash
# Chạy tất cả tests
npm test

# Chạy với watch mode
npm test -- --watch

# Chạy file cụ thể
npm test -- math.test.js

# Chạy tests matching pattern
npm test -- --testNamePattern="add"

# Chạy với coverage
npm test -- --coverage

# Chạy một test cụ thể
npm test -- -t "should return sum"
```

---

## 3. Jest Matchers

### 📊 Common Matchers

```javascript
describe("Jest Matchers", () => {
  // ═══════════════════════════════════════════
  // EQUALITY
  // ═══════════════════════════════════════════

  test("equality matchers", () => {
    // toBe: So sánh strict (===)
    expect(2 + 2).toBe(4);
    expect("hello").toBe("hello");

    // toEqual: So sánh deep equality (cho objects/arrays)
    expect({ a: 1 }).toEqual({ a: 1 });
    expect([1, 2, 3]).toEqual([1, 2, 3]);

    // toStrictEqual: Như toEqual nhưng check undefined properties
    expect({ a: 1, b: undefined }).not.toStrictEqual({ a: 1 });
  });

  // ═══════════════════════════════════════════
  // TRUTHINESS
  // ═══════════════════════════════════════════

  test("truthiness matchers", () => {
    expect(null).toBeNull();
    expect(undefined).toBeUndefined();
    expect("hello").toBeDefined();

    expect(true).toBeTruthy();
    expect(1).toBeTruthy();
    expect("hello").toBeTruthy();

    expect(false).toBeFalsy();
    expect(0).toBeFalsy();
    expect("").toBeFalsy();
    expect(null).toBeFalsy();
  });

  // ═══════════════════════════════════════════
  // NUMBERS
  // ═══════════════════════════════════════════

  test("number matchers", () => {
    expect(10).toBeGreaterThan(5);
    expect(10).toBeGreaterThanOrEqual(10);
    expect(5).toBeLessThan(10);
    expect(5).toBeLessThanOrEqual(5);

    // Floating point - dùng toBeCloseTo thay vì toBe
    expect(0.1 + 0.2).toBeCloseTo(0.3);
    expect(0.1 + 0.2).not.toBe(0.3); // Fails do floating point
  });

  // ═══════════════════════════════════════════
  // STRINGS
  // ═══════════════════════════════════════════

  test("string matchers", () => {
    expect("Hello World").toMatch(/World/);
    expect("Hello World").toMatch("World");
    expect("team").not.toMatch(/I/);

    expect("Hello").toHaveLength(5);
    expect("Hello World").toContain("World");
  });

  // ═══════════════════════════════════════════
  // ARRAYS & ITERABLES
  // ═══════════════════════════════════════════

  test("array matchers", () => {
    const fruits = ["apple", "banana", "orange"];

    expect(fruits).toContain("banana");
    expect(fruits).toHaveLength(3);
    expect(new Set(fruits)).toContain("apple");

    // Check array contains object
    const users = [{ name: "John" }, { name: "Jane" }];
    expect(users).toContainEqual({ name: "John" });

    // Check array contains subset
    expect([1, 2, 3, 4, 5]).toEqual(expect.arrayContaining([2, 4]));
  });

  // ═══════════════════════════════════════════
  // OBJECTS
  // ═══════════════════════════════════════════

  test("object matchers", () => {
    const user = {
      name: "John",
      age: 30,
      email: "john@example.com",
    };

    expect(user).toHaveProperty("name");
    expect(user).toHaveProperty("name", "John");
    expect(user).toHaveProperty("age", 30);

    // Check object contains subset
    expect(user).toMatchObject({
      name: "John",
      age: 30,
    });

    // Check object structure
    expect(user).toEqual(
      expect.objectContaining({
        name: expect.any(String),
        age: expect.any(Number),
      })
    );
  });

  // ═══════════════════════════════════════════
  // EXCEPTIONS
  // ═══════════════════════════════════════════

  test("exception matchers", () => {
    const throwError = () => {
      throw new Error("Something went wrong");
    };

    expect(throwError).toThrow();
    expect(throwError).toThrow(Error);
    expect(throwError).toThrow("Something went wrong");
    expect(throwError).toThrow(/wrong/);
  });

  // ═══════════════════════════════════════════
  // NOT (Negation)
  // ═══════════════════════════════════════════

  test("not modifier", () => {
    expect(5).not.toBe(10);
    expect([1, 2]).not.toContain(3);
    expect({ a: 1 }).not.toEqual({ a: 2 });
  });
});
```

### 🔧 Custom Matchers

```javascript
// setupTests.js
expect.extend({
  toBeWithinRange(received, floor, ceiling) {
    const pass = received >= floor && received <= ceiling;

    if (pass) {
      return {
        message: () => `expected ${received} not to be within range ${floor} - ${ceiling}`,
        pass: true,
      };
    } else {
      return {
        message: () => `expected ${received} to be within range ${floor} - ${ceiling}`,
        pass: false,
      };
    }
  },

  toBeValidEmail(received) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    const pass = emailRegex.test(received);

    return {
      message: () => `expected ${received} ${pass ? "not " : ""}to be a valid email`,
      pass,
    };
  },
});

// Sử dụng
test("custom matchers", () => {
  expect(50).toBeWithinRange(1, 100);
  expect("test@example.com").toBeValidEmail();
  expect("invalid-email").not.toBeValidEmail();
});
```

---

## 4. Mocking trong Jest

### 🎯 Tại sao cần Mock?

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        TẠI SAO CẦN MOCK?                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. ISOLATION        Test unit độc lập, không phụ thuộc dependencies   │
│  2. SPEED            Không cần gọi API/DB thật → tests chạy nhanh      │
│  3. DETERMINISTIC    Kết quả luôn giống nhau, không random             │
│  4. EDGE CASES       Dễ dàng test error cases, edge cases              │
│  5. NO SIDE EFFECTS  Không ảnh hưởng data thật                         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 💻 Mock Functions

```javascript
describe("Mock Functions", () => {
  // ═══════════════════════════════════════════
  // jest.fn() - Tạo mock function
  // ═══════════════════════════════════════════

  test("basic mock function", () => {
    const mockFn = jest.fn();

    mockFn("arg1", "arg2");
    mockFn("arg3");

    // Kiểm tra function được gọi
    expect(mockFn).toHaveBeenCalled();
    expect(mockFn).toHaveBeenCalledTimes(2);
    expect(mockFn).toHaveBeenCalledWith("arg1", "arg2");
    expect(mockFn).toHaveBeenLastCalledWith("arg3");

    // Xem tất cả các lần gọi
    expect(mockFn.mock.calls).toEqual([["arg1", "arg2"], ["arg3"]]);
  });

  // ═══════════════════════════════════════════
  // Mock Return Values
  // ═══════════════════════════════════════════

  test("mock return values", () => {
    const mockFn = jest.fn();

    // Return value cố định
    mockFn.mockReturnValue(42);
    expect(mockFn()).toBe(42);

    // Return value một lần
    mockFn.mockReturnValueOnce(1).mockReturnValueOnce(2);
    expect(mockFn()).toBe(1);
    expect(mockFn()).toBe(2);
    expect(mockFn()).toBe(42); // Fallback to mockReturnValue
  });

  // ═══════════════════════════════════════════
  // Mock Implementation
  // ═══════════════════════════════════════════

  test("mock implementation", () => {
    const mockFn = jest.fn((x) => x * 2);

    expect(mockFn(5)).toBe(10);
    expect(mockFn(3)).toBe(6);

    // Override implementation once
    mockFn.mockImplementationOnce((x) => x + 100);
    expect(mockFn(5)).toBe(105);
    expect(mockFn(5)).toBe(10); // Back to original
  });

  // ═══════════════════════════════════════════
  // Mock Async Functions
  // ═══════════════════════════════════════════

  test("mock async functions", async () => {
    const mockAsync = jest.fn();

    mockAsync.mockResolvedValue({ data: "success" });
    const result = await mockAsync();
    expect(result).toEqual({ data: "success" });

    // Mock rejected promise
    mockAsync.mockRejectedValueOnce(new Error("Failed"));
    await expect(mockAsync()).rejects.toThrow("Failed");
  });
});
```

### 📦 Mock Modules

```javascript
// userService.js
import axios from "axios";

export async function getUser(id) {
  const response = await axios.get(`/api/users/${id}`);
  return response.data;
}

export async function createUser(userData) {
  const response = await axios.post("/api/users", userData);
  return response.data;
}
```

```javascript
// userService.test.js
import axios from "axios";
import { getUser, createUser } from "./userService";

// Mock toàn bộ module axios
jest.mock("axios");

describe("User Service", () => {
  beforeEach(() => {
    // Clear mock data trước mỗi test
    jest.clearAllMocks();
  });

  test("getUser should fetch user by id", async () => {
    const mockUser = { id: 1, name: "John" };
    axios.get.mockResolvedValue({ data: mockUser });

    const user = await getUser(1);

    expect(axios.get).toHaveBeenCalledWith("/api/users/1");
    expect(user).toEqual(mockUser);
  });

  test("createUser should post user data", async () => {
    const newUser = { name: "Jane", email: "jane@example.com" };
    const createdUser = { id: 2, ...newUser };
    axios.post.mockResolvedValue({ data: createdUser });

    const result = await createUser(newUser);

    expect(axios.post).toHaveBeenCalledWith("/api/users", newUser);
    expect(result).toEqual(createdUser);
  });

  test("getUser should handle errors", async () => {
    axios.get.mockRejectedValue(new Error("Network Error"));

    await expect(getUser(1)).rejects.toThrow("Network Error");
  });
});
```

### 🔧 Partial Mock

```javascript
// utils.js
export const formatDate = (date) => date.toISOString();
export const formatCurrency = (amount) => `$${amount.toFixed(2)}`;
export const generateId = () => Math.random().toString(36).substr(2, 9);
```

```javascript
// Chỉ mock một số functions
jest.mock("./utils", () => ({
  ...jest.requireActual("./utils"), // Giữ nguyên các functions khác
  generateId: jest.fn(() => "mock-id-123"), // Chỉ mock generateId
}));

import { formatDate, formatCurrency, generateId } from "./utils";

test("partial mock", () => {
  // formatDate và formatCurrency hoạt động bình thường
  expect(formatCurrency(10)).toBe("$10.00");

  // generateId được mock
  expect(generateId()).toBe("mock-id-123");
});
```

### ⏰ Mock Timers

```javascript
// delayedFunction.js
export function delayedGreeting(callback) {
  setTimeout(() => {
    callback("Hello!");
  }, 1000);
}

export function debounce(fn, delay) {
  let timeoutId;
  return (...args) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), delay);
  };
}
```

```javascript
// delayedFunction.test.js
import { delayedGreeting, debounce } from "./delayedFunction";

describe("Timer functions", () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  test("delayedGreeting calls callback after 1 second", () => {
    const callback = jest.fn();

    delayedGreeting(callback);

    // Callback chưa được gọi
    expect(callback).not.toHaveBeenCalled();

    // Fast-forward 1 second
    jest.advanceTimersByTime(1000);

    // Callback đã được gọi
    expect(callback).toHaveBeenCalledWith("Hello!");
  });

  test("debounce should delay function call", () => {
    const fn = jest.fn();
    const debouncedFn = debounce(fn, 500);

    // Gọi nhiều lần liên tiếp
    debouncedFn("a");
    debouncedFn("b");
    debouncedFn("c");

    // Function chưa được gọi
    expect(fn).not.toHaveBeenCalled();

    // Fast-forward 500ms
    jest.advanceTimersByTime(500);

    // Chỉ gọi một lần với argument cuối cùng
    expect(fn).toHaveBeenCalledTimes(1);
    expect(fn).toHaveBeenCalledWith("c");
  });

  test("runAllTimers", () => {
    const callback = jest.fn();

    setTimeout(callback, 1000);
    setTimeout(callback, 2000);
    setTimeout(callback, 3000);

    // Chạy tất cả timers
    jest.runAllTimers();

    expect(callback).toHaveBeenCalledTimes(3);
  });
});
```

---

## 5. Testing Async Code

### 💻 Các cách test Async

```javascript
// asyncFunctions.js
export function fetchData(callback) {
  setTimeout(() => {
    callback({ data: "peanut butter" });
  }, 100);
}

export function fetchDataPromise() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({ data: "peanut butter" });
    }, 100);
  });
}

export async function fetchDataAsync() {
  const response = await fetch("/api/data");
  return response.json();
}
```

```javascript
// asyncFunctions.test.js
import { fetchData, fetchDataPromise, fetchDataAsync } from "./asyncFunctions";

describe("Async Testing", () => {
  // ═══════════════════════════════════════════
  // Callback style (dùng done)
  // ═══════════════════════════════════════════

  test("callback - using done", (done) => {
    fetchData((data) => {
      try {
        expect(data).toEqual({ data: "peanut butter" });
        done(); // Báo Jest test đã hoàn thành
      } catch (error) {
        done(error); // Báo Jest test failed
      }
    });
  });

  // ═══════════════════════════════════════════
  // Promise style
  // ═══════════════════════════════════════════

  test("promise - return promise", () => {
    // PHẢI return promise
    return fetchDataPromise().then((data) => {
      expect(data).toEqual({ data: "peanut butter" });
    });
  });

  test("promise - using resolves", () => {
    return expect(fetchDataPromise()).resolves.toEqual({ data: "peanut butter" });
  });

  test("promise - using rejects", () => {
    const failingPromise = () => Promise.reject(new Error("Failed"));
    return expect(failingPromise()).rejects.toThrow("Failed");
  });

  // ═══════════════════════════════════════════
  // Async/Await style (RECOMMENDED)
  // ═══════════════════════════════════════════

  test("async/await", async () => {
    const data = await fetchDataPromise();
    expect(data).toEqual({ data: "peanut butter" });
  });

  test("async/await with resolves", async () => {
    await expect(fetchDataPromise()).resolves.toEqual({ data: "peanut butter" });
  });

  test("async/await error handling", async () => {
    const failingAsync = async () => {
      throw new Error("Async error");
    };

    await expect(failingAsync()).rejects.toThrow("Async error");
  });

  // ═══════════════════════════════════════════
  // Mock fetch API
  // ═══════════════════════════════════════════

  test("mock fetch", async () => {
    global.fetch = jest.fn(() =>
      Promise.resolve({
        json: () => Promise.resolve({ data: "mocked data" }),
      })
    );

    const result = await fetchDataAsync();

    expect(fetch).toHaveBeenCalledWith("/api/data");
    expect(result).toEqual({ data: "mocked data" });

    // Cleanup
    global.fetch.mockRestore();
  });
});
```

### ⏱️ Timeout

```javascript
// Test với timeout dài hơn default (5000ms)
test("long running test", async () => {
  const result = await longRunningOperation();
  expect(result).toBeDefined();
}, 10000); // 10 seconds timeout

// Hoặc set global timeout
jest.setTimeout(10000);
```

---

## 6. React Testing Library

### 🎯 Philosophy

```
┌─────────────────────────────────────────────────────────────────────────┐
│              REACT TESTING LIBRARY PHILOSOPHY                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  "The more your tests resemble the way your software is used,          │
│   the more confidence they can give you."                              │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                                                                 │   │
│  │   ❌ Test implementation details                                │   │
│  │      • State values                                             │   │
│  │      • Component instances                                      │   │
│  │      • Internal methods                                         │   │
│  │                                                                 │   │
│  │   ✅ Test behavior (như user thật)                              │   │
│  │      • Rendered output                                          │   │
│  │      • User interactions                                        │   │
│  │      • Accessibility                                            │   │
│  │                                                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 🔧 Setup

```bash
npm install --save-dev @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

```javascript
// setupTests.js
import "@testing-library/jest-dom";
```

```javascript
// jest.config.js
module.exports = {
  setupFilesAfterEnv: ["<rootDir>/setupTests.js"],
  testEnvironment: "jsdom",
};
```

### 📊 Queries Priority

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    QUERY PRIORITY (Nên dùng theo thứ tự)               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. getByRole          ← BEST: Accessible cho mọi người                │
│     getByRole('button', { name: /submit/i })                           │
│                                                                         │
│  2. getByLabelText     ← Form fields                                   │
│     getByLabelText('Username')                                         │
│                                                                         │
│  3. getByPlaceholderText                                               │
│     getByPlaceholderText('Enter email')                                │
│                                                                         │
│  4. getByText          ← Non-interactive elements                      │
│     getByText('Hello World')                                           │
│                                                                         │
│  5. getByDisplayValue  ← Current value of form elements                │
│     getByDisplayValue('current value')                                 │
│                                                                         │
│  6. getByAltText       ← Images                                        │
│     getByAltText('profile picture')                                    │
│                                                                         │
│  7. getByTitle                                                         │
│     getByTitle('Close')                                                │
│                                                                         │
│  8. getByTestId        ← LAST RESORT                                   │
│     getByTestId('custom-element')                                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 💻 Query Types

```javascript
import { render, screen } from "@testing-library/react";

// ═══════════════════════════════════════════
// getBy* - Throws error nếu không tìm thấy
// ═══════════════════════════════════════════
const button = screen.getByRole("button");
const heading = screen.getByText("Hello");

// ═══════════════════════════════════════════
// queryBy* - Returns null nếu không tìm thấy
// ═══════════════════════════════════════════
const maybeButton = screen.queryByRole("button");
expect(maybeButton).not.toBeInTheDocument();

// ═══════════════════════════════════════════
// findBy* - Returns Promise, dùng cho async
// ═══════════════════════════════════════════
const asyncElement = await screen.findByText("Loaded");

// ═══════════════════════════════════════════
// *AllBy* - Returns array
// ═══════════════════════════════════════════
const buttons = screen.getAllByRole("button");
const items = screen.queryAllByRole("listitem");
const asyncItems = await screen.findAllByRole("listitem");
```

### 📋 Query Cheat Sheet

| Query        | No Match | 1 Match | >1 Match | Await? |
| ------------ | -------- | ------- | -------- | ------ |
| `getBy`      | throw    | return  | throw    | No     |
| `queryBy`    | null     | return  | throw    | No     |
| `findBy`     | throw    | return  | throw    | Yes    |
| `getAllBy`   | throw    | array   | array    | No     |
| `queryAllBy` | []       | array   | array    | No     |
| `findAllBy`  | throw    | array   | array    | Yes    |

---

## 7. Testing React Components

### 💻 Basic Component Testing

```jsx
// Button.jsx
export function Button({ onClick, children, disabled = false, variant = "primary" }) {
  return (
    <button onClick={onClick} disabled={disabled} className={`btn btn-${variant}`}>
      {children}
    </button>
  );
}
```

```jsx
// Button.test.jsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { Button } from "./Button";

describe("Button", () => {
  test("renders button with text", () => {
    render(<Button>Click me</Button>);

    expect(screen.getByRole("button", { name: /click me/i })).toBeInTheDocument();
  });

  test("calls onClick when clicked", async () => {
    const handleClick = jest.fn();
    const user = userEvent.setup();

    render(<Button onClick={handleClick}>Click me</Button>);

    await user.click(screen.getByRole("button"));

    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  test("does not call onClick when disabled", async () => {
    const handleClick = jest.fn();
    const user = userEvent.setup();

    render(
      <Button onClick={handleClick} disabled>
        Click me
      </Button>
    );

    await user.click(screen.getByRole("button"));

    expect(handleClick).not.toHaveBeenCalled();
  });

  test("applies variant class", () => {
    render(<Button variant="secondary">Click me</Button>);

    expect(screen.getByRole("button")).toHaveClass("btn-secondary");
  });

  test("is disabled when disabled prop is true", () => {
    render(<Button disabled>Click me</Button>);

    expect(screen.getByRole("button")).toBeDisabled();
  });
});
```

### 🔧 Testing Component với State

```jsx
// Counter.jsx
import { useState } from "react";

export function Counter({ initialCount = 0 }) {
  const [count, setCount] = useState(initialCount);

  return (
    <div>
      <span data-testid="count">Count: {count}</span>
      <button onClick={() => setCount((c) => c - 1)}>Decrement</button>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

```jsx
// Counter.test.jsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { Counter } from "./Counter";

describe("Counter", () => {
  test("renders with initial count", () => {
    render(<Counter initialCount={5} />);

    expect(screen.getByText("Count: 5")).toBeInTheDocument();
  });

  test("increments count when increment button is clicked", async () => {
    const user = userEvent.setup();
    render(<Counter />);

    await user.click(screen.getByRole("button", { name: /increment/i }));

    expect(screen.getByText("Count: 1")).toBeInTheDocument();
  });

  test("decrements count when decrement button is clicked", async () => {
    const user = userEvent.setup();
    render(<Counter initialCount={5} />);

    await user.click(screen.getByRole("button", { name: /decrement/i }));

    expect(screen.getByText("Count: 4")).toBeInTheDocument();
  });

  test("resets count when reset button is clicked", async () => {
    const user = userEvent.setup();
    render(<Counter initialCount={10} />);

    await user.click(screen.getByRole("button", { name: /increment/i }));
    await user.click(screen.getByRole("button", { name: /reset/i }));

    expect(screen.getByText("Count: 0")).toBeInTheDocument();
  });
});
```

### 🔄 Testing Conditional Rendering

```jsx
// UserGreeting.jsx
export function UserGreeting({ user, isLoading, error }) {
  if (isLoading) {
    return <div role="status">Loading...</div>;
  }

  if (error) {
    return <div role="alert">Error: {error}</div>;
  }

  if (!user) {
    return <div>Please log in</div>;
  }

  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      <p>Email: {user.email}</p>
    </div>
  );
}
```

```jsx
// UserGreeting.test.jsx
import { render, screen } from "@testing-library/react";
import { UserGreeting } from "./UserGreeting";

describe("UserGreeting", () => {
  test("shows loading state", () => {
    render(<UserGreeting isLoading={true} />);

    expect(screen.getByRole("status")).toHaveTextContent("Loading...");
  });

  test("shows error message", () => {
    render(<UserGreeting error="Something went wrong" />);

    expect(screen.getByRole("alert")).toHaveTextContent("Error: Something went wrong");
  });

  test("shows login prompt when no user", () => {
    render(<UserGreeting />);

    expect(screen.getByText("Please log in")).toBeInTheDocument();
  });

  test("shows user info when user is provided", () => {
    const user = { name: "John", email: "john@example.com" };
    render(<UserGreeting user={user} />);

    expect(screen.getByRole("heading")).toHaveTextContent("Welcome, John!");
    expect(screen.getByText("Email: john@example.com")).toBeInTheDocument();
  });

  test("loading takes precedence over user", () => {
    const user = { name: "John", email: "john@example.com" };
    render(<UserGreeting user={user} isLoading={true} />);

    expect(screen.getByRole("status")).toBeInTheDocument();
    expect(screen.queryByRole("heading")).not.toBeInTheDocument();
  });
});
```

---

## 8. Testing Hooks

### 🔧 Testing Custom Hooks

```jsx
// useCounter.js
import { useState, useCallback } from "react";

export function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => setCount((c) => c + 1), []);
  const decrement = useCallback(() => setCount((c) => c - 1), []);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);

  return { count, increment, decrement, reset };
}
```

```jsx
// useCounter.test.js
import { renderHook, act } from "@testing-library/react";
import { useCounter } from "./useCounter";

describe("useCounter", () => {
  test("should initialize with default value", () => {
    const { result } = renderHook(() => useCounter());

    expect(result.current.count).toBe(0);
  });

  test("should initialize with provided value", () => {
    const { result } = renderHook(() => useCounter(10));

    expect(result.current.count).toBe(10);
  });

  test("should increment count", () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });

  test("should decrement count", () => {
    const { result } = renderHook(() => useCounter(5));

    act(() => {
      result.current.decrement();
    });

    expect(result.current.count).toBe(4);
  });

  test("should reset count", () => {
    const { result } = renderHook(() => useCounter(10));

    act(() => {
      result.current.increment();
      result.current.increment();
      result.current.reset();
    });

    expect(result.current.count).toBe(10);
  });

  test("should update when initialValue changes", () => {
    const { result, rerender } = renderHook(({ initialValue }) => useCounter(initialValue), {
      initialProps: { initialValue: 0 },
    });

    expect(result.current.count).toBe(0);

    rerender({ initialValue: 100 });

    act(() => {
      result.current.reset();
    });

    expect(result.current.count).toBe(100);
  });
});
```

### 🔧 Testing Async Hooks

```jsx
// useFetch.js
import { useState, useEffect } from "react";

export function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchData() {
      try {
        setLoading(true);
        const response = await fetch(url);

        if (!response.ok) {
          throw new Error("Failed to fetch");
        }

        const json = await response.json();

        if (!cancelled) {
          setData(json);
          setError(null);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err.message);
          setData(null);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }

    fetchData();

    return () => {
      cancelled = true;
    };
  }, [url]);

  return { data, loading, error };
}
```

```jsx
// useFetch.test.js
import { renderHook, waitFor } from "@testing-library/react";
import { useFetch } from "./useFetch";

describe("useFetch", () => {
  beforeEach(() => {
    global.fetch = jest.fn();
  });

  afterEach(() => {
    jest.resetAllMocks();
  });

  test("should start with loading state", () => {
    global.fetch.mockImplementation(() => new Promise(() => {}));

    const { result } = renderHook(() => useFetch("/api/data"));

    expect(result.current.loading).toBe(true);
    expect(result.current.data).toBeNull();
    expect(result.current.error).toBeNull();
  });

  test("should fetch data successfully", async () => {
    const mockData = { id: 1, name: "Test" };
    global.fetch.mockResolvedValue({
      ok: true,
      json: () => Promise.resolve(mockData),
    });

    const { result } = renderHook(() => useFetch("/api/data"));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.data).toEqual(mockData);
    expect(result.current.error).toBeNull();
  });

  test("should handle fetch error", async () => {
    global.fetch.mockResolvedValue({
      ok: false,
    });

    const { result } = renderHook(() => useFetch("/api/data"));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.data).toBeNull();
    expect(result.current.error).toBe("Failed to fetch");
  });

  test("should refetch when url changes", async () => {
    const mockData1 = { id: 1 };
    const mockData2 = { id: 2 };

    global.fetch
      .mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve(mockData1),
      })
      .mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve(mockData2),
      });

    const { result, rerender } = renderHook(({ url }) => useFetch(url), { initialProps: { url: "/api/data/1" } });

    await waitFor(() => {
      expect(result.current.data).toEqual(mockData1);
    });

    rerender({ url: "/api/data/2" });

    await waitFor(() => {
      expect(result.current.data).toEqual(mockData2);
    });
  });
});
```

---

## 9. Testing Forms & User Events

### 💻 Testing Form Input

```jsx
// LoginForm.jsx
import { useState } from 'react';

export function LoginForm({ onSubmit }) {

```

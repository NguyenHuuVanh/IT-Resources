# Iterators & Generators

> Quay lại [Mục lục](./README.md) | Trước: [Symbol](./11-symbol.md)

## 📌 Tổng quan

**Iterators** và **Generators** là hai khái niệm liên quan cho phép:
- **Iterator**: Định nghĩa cách duyệt tuần tự qua một tập dữ liệu
- **Generator**: Tạo iterator một cách dễ dàng hơn bằng `function*` và `yield`

---

## 1. Iterator Protocol

### 1.1. Khái niệm

Một object là **iterator** khi nó implement method `next()` trả về `{ value, done }`.

```javascript
// Manual iterator
const iterator = {
  current: 1,
  last: 5,

  next() {
    if (this.current <= this.last) {
      return { value: this.current++, done: false };
    }
    return { value: undefined, done: true };
  },
};

console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
console.log(iterator.next()); // { value: 3, done: false }
console.log(iterator.next()); // { value: 4, done: false }
console.log(iterator.next()); // { value: 5, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

### 1.2. Iterable Protocol

Object là **iterable** khi nó implement `[Symbol.iterator]()` method trả về iterator.

```javascript
const iterable = {
  data: [10, 20, 30],

  [Symbol.iterator]() {
    let index = 0;
    const data = this.data;

    return {
      next() {
        if (index < data.length) {
          return { value: data[index++], done: false };
        }
        return { done: true };
      },
    };
  },
};

// Giờ có thể dùng for...of
for (const item of iterable) {
  console.log(item); // 10, 20, 30
}

// Và spread
console.log([...iterable]); // [10, 20, 30]

// Và destructuring
const [first, second] = iterable; // first=10, second=20
```

### 1.3. Built-in Iterables

Các kiểu dữ liệu built-in đã implement `[Symbol.iterator]`:

```javascript
// String
for (const char of "hello") {
  console.log(char); // "h", "e", "l", "l", "o"
}

// Array
for (const item of [1, 2, 3]) {
  console.log(item);
}

// Map
for (const [key, value] of new Map([["a", 1], ["b", 2]])) {
  console.log(key, value);
}

// Set
for (const value of new Set([1, 2, 3])) {
  console.log(value);
}

// TypedArray, NodeList, arguments, etc.
```

### 1.4. Custom Iterable — Range

```javascript
class Range {
  constructor(start, end, step = 1) {
    this.start = start;
    this.end = end;
    this.step = step;
  }

  [Symbol.iterator]() {
    let current = this.start;
    const { end, step } = this;

    return {
      next() {
        if (current <= end) {
          const value = current;
          current += step;
          return { value, done: false };
        }
        return { done: true };
      },

      // Optional: make iterator itself iterable
      [Symbol.iterator]() {
        return this;
      },
    };
  }
}

const range = new Range(1, 10, 2);
console.log([...range]); // [1, 3, 5, 7, 9]

for (const n of new Range(0, 5)) {
  console.log(n); // 0, 1, 2, 3, 4, 5
}
```

---

## 2. Generator Functions

### 2.1. Cú pháp cơ bản

```javascript
// Generator function (function*)
function* numberGenerator() {
  yield 1;    // Tạm dừng, trả về 1
  yield 2;    // Tạm dừng, trả về 2
  yield 3;    // Tạm dừng, trả về 3
  return 4;   // Kết thúc, trả về 4 (done: true)
}

const gen = numberGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: 4, done: true }
console.log(gen.next()); // { value: undefined, done: true }

// Generator tự động implement iterator protocol
for (const num of numberGenerator()) {
  console.log(num); // 1, 2, 3 (⚠️ return value không included!)
}

console.log([...numberGenerator()]); // [1, 2, 3]
```

### 2.2. Generator với Logic

```javascript
// Range generator
function* range(start, end, step = 1) {
  for (let i = start; i <= end; i += step) {
    yield i;
  }
}

console.log([...range(1, 10)]);      // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
console.log([...range(0, 100, 20)]); // [0, 20, 40, 60, 80, 100]

// Fibonacci generator
function* fibonacci() {
  let [a, b] = [0, 1];
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

// Lấy 10 số Fibonacci đầu tiên
const fib = fibonacci();
const first10 = Array.from({ length: 10 }, () => fib.next().value);
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

### 2.3. Infinite Generators

```javascript
// Infinite sequence
function* infiniteCounter(start = 0) {
  let count = start;
  while (true) {
    yield count++;
  }
}

const counter = infiniteCounter(1);
console.log(counter.next().value); // 1
console.log(counter.next().value); // 2
console.log(counter.next().value); // 3
// ... tiếp tục mãi mãi

// ⚠️ ĐỪNG dùng [...infiniteGen()] — infinite loop!

// Take utility
function* take(n, iterable) {
  let count = 0;
  for (const item of iterable) {
    if (count >= n) return;
    yield item;
    count++;
  }
}

console.log([...take(5, infiniteCounter())]); // [0, 1, 2, 3, 4]
```

---

## 3. `yield*` — Generator Delegation

### 3.1. Delegate to Another Generator

```javascript
function* gen1() {
  yield 1;
  yield 2;
}

function* gen2() {
  yield* gen1();  // Delegate — yield all values from gen1
  yield 3;
  yield 4;
}

console.log([...gen2()]); // [1, 2, 3, 4]

// Tương đương:
function* gen2Manual() {
  for (const value of gen1()) {
    yield value;
  }
  yield 3;
  yield 4;
}
```

### 3.2. Delegate to Any Iterable

```javascript
function* combined() {
  yield* [1, 2, 3];        // Array
  yield* "abc";              // String
  yield* new Set([4, 5]);   // Set
}

console.log([...combined()]); // [1, 2, 3, "a", "b", "c", 4, 5]
```

### 3.3. Recursive Generator — Flatten

```javascript
function* flatten(arr) {
  for (const item of arr) {
    if (Array.isArray(item)) {
      yield* flatten(item); // Recursive delegation
    } else {
      yield item;
    }
  }
}

const nested = [1, [2, [3, [4, [5]]]]];
console.log([...flatten(nested)]); // [1, 2, 3, 4, 5]

// Tree traversal
function* walkTree(node) {
  yield node.value;
  if (node.children) {
    for (const child of node.children) {
      yield* walkTree(child);
    }
  }
}
```

---

## 4. Two-way Communication

### 4.1. Passing Values IN (via `.next(value)`)

```javascript
function* conversation() {
  const name = yield "What is your name?";
  const age = yield `Hello ${name}! How old are you?`;
  return `${name} is ${age} years old`;
}

const talk = conversation();
console.log(talk.next().value);       // "What is your name?"
console.log(talk.next("John").value); // "Hello John! How old are you?"
console.log(talk.next(25).value);     // "John is 25 years old"

// Flow:
// 1. next()      → chạy đến yield đầu tiên → return "What is your name?"
// 2. next("John") → "John" gán vào name → chạy đến yield 2
// 3. next(25)     → 25 gán vào age → chạy đến return
```

### 4.2. `.throw()` — Throw Error into Generator

```javascript
function* safeDivide() {
  try {
    const a = yield "Enter first number";
    const b = yield "Enter second number";
    
    if (b === 0) {
      throw new Error("Cannot divide by zero");
    }
    
    yield a / b;
  } catch (error) {
    yield `Error: ${error.message}`;
  }
}

const gen = safeDivide();
gen.next();          // "Enter first number"
gen.next(10);        // "Enter second number"
gen.throw(new Error("User cancelled")); // "Error: User cancelled"
```

### 4.3. `.return()` — Force Return

```javascript
function* counter() {
  let count = 0;
  while (true) {
    yield count++;
  }
}

const gen = counter();
gen.next();         // { value: 0, done: false }
gen.next();         // { value: 1, done: false }
gen.return("end");  // { value: "end", done: true } — Force stop
gen.next();         // { value: undefined, done: true }
```

---

## 5. Practical Examples

### 5.1. ID Generator

```javascript
function* idGenerator(prefix = "id") {
  let id = 1;
  while (true) {
    yield `${prefix}_${String(id++).padStart(6, "0")}`;
  }
}

const userIds = idGenerator("user");
console.log(userIds.next().value); // "user_000001"
console.log(userIds.next().value); // "user_000002"

const orderIds = idGenerator("order");
console.log(orderIds.next().value); // "order_000001"
```

### 5.2. Pagination

```javascript
function* paginate(items, pageSize) {
  for (let i = 0; i < items.length; i += pageSize) {
    yield {
      page: Math.floor(i / pageSize) + 1,
      data: items.slice(i, i + pageSize),
      hasNext: i + pageSize < items.length,
    };
  }
}

const items = Array.from({ length: 25 }, (_, i) => `Item ${i + 1}`);
const pages = paginate(items, 10);

console.log(pages.next().value);
// { page: 1, data: ["Item 1", ..., "Item 10"], hasNext: true }

console.log(pages.next().value);
// { page: 2, data: ["Item 11", ..., "Item 20"], hasNext: true }

console.log(pages.next().value);
// { page: 3, data: ["Item 21", ..., "Item 25"], hasNext: false }
```

### 5.3. State Machine

```javascript
function* trafficLight() {
  while (true) {
    yield "🟢 GREEN";
    yield "🟡 YELLOW";
    yield "🔴 RED";
  }
}

const light = trafficLight();
console.log(light.next().value); // 🟢 GREEN
console.log(light.next().value); // 🟡 YELLOW
console.log(light.next().value); // 🔴 RED
console.log(light.next().value); // 🟢 GREEN (lặp lại)
```

### 5.4. Lazy Evaluation Pipeline

```javascript
function* filter(predicate, iterable) {
  for (const item of iterable) {
    if (predicate(item)) yield item;
  }
}

function* map(transform, iterable) {
  for (const item of iterable) {
    yield transform(item);
  }
}

function* take(n, iterable) {
  let count = 0;
  for (const item of iterable) {
    if (count >= n) return;
    yield item;
    count++;
  }
}

// Pipeline — lazy, chỉ compute khi cần
function* naturals() {
  let n = 1;
  while (true) yield n++;
}

const result = take(5, 
  map(x => x * x, 
    filter(x => x % 2 === 0, 
      naturals()
    )
  )
);

console.log([...result]); // [4, 16, 36, 64, 100]
// Chỉ xử lý đủ 5 số chẵn bình phương, không xử lý vô hạn
```

---

## 6. Async Generators (ES2018)

```javascript
// Async generator function
async function* fetchPages(baseUrl) {
  let page = 1;
  let hasMore = true;

  while (hasMore) {
    const response = await fetch(`${baseUrl}?page=${page}`);
    const data = await response.json();
    
    yield data.items;
    
    hasMore = data.hasNextPage;
    page++;
  }
}

// for await...of
async function processAllPages() {
  for await (const items of fetchPages("/api/users")) {
    for (const item of items) {
      processItem(item);
    }
  }
}
```

---

## 7. Câu hỏi phỏng vấn

### Q1: Iterator khác Generator thế nào?
**A**: Iterator là protocol (interface) — bất kỳ object nào có `.next()` trả về `{value, done}`. Generator là function đặc biệt (`function*`) giúp TẠO iterator một cách dễ dàng bằng `yield`.

### Q2: `yield` khác `return` thế nào?
**A**: `yield` tạm dừng generator và có thể resume. `return` kết thúc generator vĩnh viễn. `yield` value included trong `for...of`, `return` value thì không.

### Q3: Generator có lazy evaluation không?
**A**: Có. Values chỉ được tính khi `.next()` được gọi, cho phép xử lý infinite sequences và tiết kiệm memory.

---

> Tiếp theo: [Map & Set →](./13-map-set.md)

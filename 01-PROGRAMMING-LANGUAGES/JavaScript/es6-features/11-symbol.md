# Symbol

> Quay lại [Mục lục](./README.md) | Trước: [Promises](./10-promises.md)

## 📌 Tổng quan

**Symbol** là kiểu dữ liệu **primitive** mới (thứ 7) trong ES6, mỗi Symbol là **unique** (duy nhất) và **immutable** (bất biến). Mục đích chính là tạo **property keys duy nhất** cho objects, tránh xung đột tên.

### 7 Primitive Types trong JavaScript

```
string, number, boolean, null, undefined, symbol (ES6), bigint (ES2020)
```

---

## 1. Tạo Symbol

```javascript
// Tạo Symbol (KHÔNG dùng new)
const sym1 = Symbol();
const sym2 = Symbol("description"); // description chỉ để debug
const sym3 = Symbol("description");

// ⚠️ Mỗi Symbol là UNIQUE
console.log(sym2 === sym3); // false (dù cùng description!)
console.log(sym1 === sym2); // false

// typeof
console.log(typeof sym1); // "symbol"

// ❌ KHÔNG thể convert tự động
// sym1 + "";      // TypeError: Cannot convert a Symbol to a string
// +sym1;          // TypeError: Cannot convert a Symbol to a number
// `${sym1}`;      // TypeError

// ✅ Convert thủ công
console.log(sym1.toString());    // "Symbol()"
console.log(sym2.toString());    // "Symbol(description)"
console.log(sym2.description);   // "description" (ES2019)
console.log(String(sym1));       // "Symbol()"
```

---

## 2. Symbol as Object Keys

### 2.1. Unique Property Keys

```javascript
const id = Symbol("id");
const name = Symbol("name");

const user = {
  [id]: 12345,           // Symbol key
  [name]: "John",        // Symbol key
  age: 25,               // String key
  email: "j@x.com",     // String key
};

// Access
console.log(user[id]);    // 12345
console.log(user[name]);  // "John"

// ⚠️ Không thể dùng dot notation
// user.id      → truy cập string key "id", KHÔNG phải Symbol
// user[id]     → truy cập Symbol key ✅
```

### 2.2. Symbol Keys ẩn khỏi enumeration

```javascript
const secret = Symbol("secret");
const obj = {
  name: "public",
  [secret]: "hidden",
};

// ❌ KHÔNG xuất hiện trong:
console.log(Object.keys(obj));              // ["name"]
console.log(Object.values(obj));            // ["public"]
console.log(Object.entries(obj));           // [["name", "public"]]
console.log(JSON.stringify(obj));           // '{"name":"public"}'

for (let key in obj) {
  console.log(key); // Chỉ "name"
}

// ✅ Lấy Symbol keys:
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(secret)]

// ✅ Lấy TẤT CẢ keys (string + symbol):
console.log(Reflect.ownKeys(obj)); // ["name", Symbol(secret)]
```

### 2.3. Use Case: Tránh xung đột

```javascript
// Thư viện A thêm metadata
const libA_id = Symbol("id");
obj[libA_id] = "from-lib-A";

// Thư viện B cũng thêm metadata — KHÔNG xung đột!
const libB_id = Symbol("id");
obj[libB_id] = "from-lib-B";

console.log(obj[libA_id]); // "from-lib-A"
console.log(obj[libB_id]); // "from-lib-B"
// Cả hai tồn tại song song mà không ghi đè nhau
```

---

## 3. Global Symbol Registry

### 3.1. `Symbol.for()`

Tạo hoặc lấy Symbol từ **global registry** (chia sẻ giữa các modules/files).

```javascript
// Symbol.for() tạo Symbol và lưu vào global registry
const globalSym1 = Symbol.for("app.id");
const globalSym2 = Symbol.for("app.id");

// Cùng key → cùng Symbol
console.log(globalSym1 === globalSym2); // true ✅

// So sánh với Symbol() — luôn unique
const localSym1 = Symbol("app.id");
const localSym2 = Symbol("app.id");
console.log(localSym1 === localSym2); // false ❌
```

### 3.2. `Symbol.keyFor()`

```javascript
// Lấy key của global Symbol
const sym = Symbol.for("app.config");
console.log(Symbol.keyFor(sym)); // "app.config"

// Regular Symbol không có trong registry
const localSym = Symbol("local");
console.log(Symbol.keyFor(localSym)); // undefined
```

### 3.3. Use Case: Cross-module Communication

```javascript
// module-a.js
const EVENT_TYPE = Symbol.for("myapp.eventType");
export function createEvent(data) {
  return { [EVENT_TYPE]: "click", data };
}

// module-b.js
const EVENT_TYPE = Symbol.for("myapp.eventType");
export function handleEvent(event) {
  console.log(event[EVENT_TYPE]); // "click" ← cùng Symbol!
}
```

---

## 4. Well-known Symbols

JavaScript có các **built-in Symbols** cho phép customize hành vi của objects.

### 4.1. `Symbol.iterator`

Định nghĩa cách object được iterate (for...of).

```javascript
const range = {
  from: 1,
  to: 5,

  [Symbol.iterator]() {
    let current = this.from;
    const last = this.to;

    return {
      next() {
        if (current <= last) {
          return { value: current++, done: false };
        }
        return { done: true };
      },
    };
  },
};

for (const num of range) {
  console.log(num); // 1, 2, 3, 4, 5
}

console.log([...range]); // [1, 2, 3, 4, 5]
```

### 4.2. `Symbol.toStringTag`

Customize `Object.prototype.toString()`.

```javascript
class Database {
  get [Symbol.toStringTag]() {
    return "Database";
  }
}

const db = new Database();
console.log(Object.prototype.toString.call(db)); 
// "[object Database]" (thay vì "[object Object]")
console.log(`${db}`); // "[object Database]"
```

### 4.3. `Symbol.toPrimitive`

Customize type conversion.

```javascript
class Money {
  constructor(amount, currency) {
    this.amount = amount;
    this.currency = currency;
  }

  [Symbol.toPrimitive](hint) {
    switch (hint) {
      case "number":
        return this.amount;
      case "string":
        return `${this.amount} ${this.currency}`;
      case "default":
        return this.amount;
    }
  }
}

const price = new Money(100, "USD");
console.log(+price);        // 100       (hint: "number")
console.log(`${price}`);    // "100 USD" (hint: "string")
console.log(price + 50);    // 150       (hint: "default")
console.log(price > 50);    // true      (hint: "number")
```

### 4.4. `Symbol.hasInstance`

Customize `instanceof`.

```javascript
class EvenNumber {
  static [Symbol.hasInstance](num) {
    return typeof num === "number" && num % 2 === 0;
  }
}

console.log(4 instanceof EvenNumber);  // true
console.log(3 instanceof EvenNumber);  // false
console.log("4" instanceof EvenNumber); // false
```

### 4.5. `Symbol.species`

Customize constructor cho derived objects.

```javascript
class PowerArray extends Array {
  static get [Symbol.species]() {
    return Array; // Methods như map, filter trả về Array thường
  }
}

const arr = new PowerArray(1, 2, 3);
const filtered = arr.filter(x => x > 1);

console.log(filtered instanceof PowerArray); // false
console.log(filtered instanceof Array);      // true
```

### 4.6. Danh sách Well-known Symbols

| Symbol | Mục đích |
|--------|----------|
| `Symbol.iterator` | Định nghĩa iterator (for...of) |
| `Symbol.asyncIterator` | Async iterator (for await...of) |
| `Symbol.toStringTag` | Customize toString() |
| `Symbol.toPrimitive` | Type conversion |
| `Symbol.hasInstance` | Customize instanceof |
| `Symbol.species` | Constructor cho derived |
| `Symbol.match` | String.match() behavior |
| `Symbol.replace` | String.replace() behavior |
| `Symbol.search` | String.search() behavior |
| `Symbol.split` | String.split() behavior |
| `Symbol.isConcatSpreadable` | Array.concat() behavior |
| `Symbol.unscopables` | with statement exclusion |

---

## 5. Practical Use Cases

### 5.1. Enum Pattern

```javascript
const Direction = Object.freeze({
  UP: Symbol("UP"),
  DOWN: Symbol("DOWN"),
  LEFT: Symbol("LEFT"),
  RIGHT: Symbol("RIGHT"),
});

function move(direction) {
  switch (direction) {
    case Direction.UP: return "Moving up";
    case Direction.DOWN: return "Moving down";
    case Direction.LEFT: return "Moving left";
    case Direction.RIGHT: return "Moving right";
    default: throw new Error("Invalid direction");
  }
}

// Không thể giả mạo — Symbol là unique
move(Direction.UP);        // ✅ "Moving up"
move("UP");                // ❌ Error (string "UP" !== Symbol("UP"))
move(Symbol("UP"));        // ❌ Error (different Symbol)
```

### 5.2. Private-like Properties

```javascript
const _age = Symbol("age");
const _validate = Symbol("validate");

class Person {
  constructor(name, age) {
    this.name = name;        // Public
    this[_age] = age;        // "Private" (hidden from enumeration)
  }

  [_validate]() {
    return this[_age] > 0 && this[_age] < 150;
  }

  getInfo() {
    if (!this[_validate]()) throw new Error("Invalid data");
    return `${this.name}, age ${this[_age]}`;
  }
}
```

---

## 6. Câu hỏi phỏng vấn

### Q1: Symbol là gì? Tại sao cần?
**A**: Symbol là primitive type mới, mỗi giá trị là unique. Chủ yếu dùng làm object keys để tránh xung đột tên giữa các thư viện.

### Q2: `Symbol()` vs `Symbol.for()`?
**A**: `Symbol()` luôn tạo Symbol mới (unique). `Symbol.for("key")` tạo hoặc lấy Symbol từ global registry — cùng key = cùng Symbol.

### Q3: Symbol keys có xuất hiện trong `Object.keys()` không?
**A**: Không. Phải dùng `Object.getOwnPropertySymbols()` hoặc `Reflect.ownKeys()`.

---

> Tiếp theo: [Iterators & Generators →](./12-iterators-generators.md)

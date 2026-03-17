# Proxy & Reflect

> Quay lại [Mục lục](./README.md) | Trước: [New Built-in Methods](./15-new-methods.md)

## 📌 Tổng quan

**Proxy** và **Reflect** là hai tính năng **metaprogramming** trong ES6:
- **Proxy**: Tạo "wrapper" xung quanh object để **intercept** (chặn) và **customize** các thao tác cơ bản (get, set, delete, ...)
- **Reflect**: API tiêu chuẩn cung cấp các methods tương ứng với proxy traps

### Metaprogramming là gì?

Metaprogramming là kỹ thuật viết code để **thao tác trên chính code** — thay đổi cách objects hoạt động ở mức fundamental.

---

## 1. Proxy Basics

### 1.1. Cú pháp

```javascript
// const proxy = new Proxy(target, handler)
// - target: object gốc cần wrap
// - handler: object chứa "traps" (các hàm intercept)

const target = {
  name: "John",
  age: 25,
};

const handler = {
  // Trap cho property access
  get(target, prop, receiver) {
    console.log(`Getting "${prop}"`);
    return target[prop];
  },

  // Trap cho property assignment
  set(target, prop, value, receiver) {
    console.log(`Setting "${prop}" to ${value}`);
    target[prop] = value;
    return true; // ⚠️ PHẢI return true nếu thành công
  },
};

const proxy = new Proxy(target, handler);

proxy.name;      // log: Getting "name" → "John"
proxy.age = 30;  // log: Setting "age" to 30
```

### 1.2. Empty Handler (Transparent Proxy)

```javascript
// Handler rỗng — proxy hoạt động giống hệt target
const proxy = new Proxy(target, {});
proxy.name;      // "John" (pass-through)
proxy.age = 30;  // OK (pass-through)
```

---

## 2. Proxy Traps (tất cả 13 traps)

```javascript
const handler = {
  // ═══ Property operations ═══
  get(target, prop, receiver) {},           // obj.prop, obj[prop]
  set(target, prop, value, receiver) {},    // obj.prop = value
  has(target, prop) {},                     // prop in obj
  deleteProperty(target, prop) {},          // delete obj.prop
  
  // ═══ Property descriptor operations ═══
  getOwnPropertyDescriptor(target, prop) {},
  defineProperty(target, prop, descriptor) {},
  ownKeys(target) {},                       // Object.keys(), for...in, etc.
  
  // ═══ Function operations ═══
  apply(target, thisArg, args) {},          // proxy()
  construct(target, args, newTarget) {},    // new proxy()
  
  // ═══ Prototype operations ═══
  getPrototypeOf(target) {},               // Object.getPrototypeOf()
  setPrototypeOf(target, proto) {},        // Object.setPrototypeOf()
  
  // ═══ Extensibility operations ═══
  isExtensible(target) {},                 // Object.isExtensible()
  preventExtensions(target) {},            // Object.preventExtensions()
};
```

---

## 3. Common Trap Examples

### 3.1. Get Trap — Default Values, Logging

```javascript
// Default values cho properties không tồn tại
const withDefaults = (target, defaults) => {
  return new Proxy(target, {
    get(obj, prop) {
      return prop in obj ? obj[prop] : defaults[prop];
    },
  });
};

const config = withDefaults({}, {
  theme: "dark",
  language: "en",
  fontSize: 14,
});

console.log(config.theme);    // "dark" (default)
console.log(config.language); // "en" (default)
config.theme = "light";
console.log(config.theme);    // "light" (set value)
```

### 3.2. Set Trap — Validation

```javascript
const validator = {
  set(target, prop, value) {
    // Type validation
    if (prop === "age") {
      if (typeof value !== "number") {
        throw new TypeError("Age must be a number");
      }
      if (!Number.isInteger(value)) {
        throw new TypeError("Age must be an integer");
      }
      if (value < 0 || value > 150) {
        throw new RangeError("Age must be between 0 and 150");
      }
    }
    
    if (prop === "email") {
      if (typeof value !== "string" || !value.includes("@")) {
        throw new TypeError("Invalid email format");
      }
    }
    
    if (prop === "name") {
      if (typeof value !== "string" || value.length < 2) {
        throw new TypeError("Name must be at least 2 characters");
      }
      value = value.trim(); // Auto-trim
    }
    
    target[prop] = value;
    return true;
  },
};

const person = new Proxy({}, validator);
person.age = 25;           // ✅ OK
person.age = -5;           // ❌ RangeError
person.age = "old";        // ❌ TypeError
person.email = "invalid";  // ❌ TypeError
person.email = "j@x.com";  // ✅ OK
person.name = "a";         // ❌ TypeError (too short)
```

### 3.3. Has Trap — Custom `in` operator

```javascript
const range = new Proxy({ from: 1, to: 100 }, {
  has(target, prop) {
    const num = Number(prop);
    return num >= target.from && num <= target.to;
  },
});

console.log(50 in range);  // true
console.log(150 in range); // false
console.log(0 in range);   // false
```

### 3.4. Delete Trap — Prevent Deletion

```javascript
const protected = new Proxy({ id: 1, name: "John", role: "admin" }, {
  deleteProperty(target, prop) {
    if (prop === "id") {
      throw new Error("Cannot delete id property");
    }
    delete target[prop];
    return true;
  },
});

delete protected.name; // ✅ OK
delete protected.id;   // ❌ Error
```

### 3.5. Apply Trap — Function Proxy

```javascript
// Logging function calls
function createLogged(fn) {
  return new Proxy(fn, {
    apply(target, thisArg, args) {
      console.log(`Calling ${target.name}(${args.join(", ")})`);
      const start = performance.now();
      const result = target.apply(thisArg, args);
      const duration = performance.now() - start;
      console.log(`→ Returned: ${result} (${duration.toFixed(2)}ms)`);
      return result;
    },
  });
}

const add = createLogged((a, b) => a + b);
add(2, 3);
// Calling add(2, 3)
// → Returned: 5 (0.01ms)
```

---

## 4. Practical Proxy Patterns

### 4.1. Negative Array Indices

```javascript
function negativeArray(arr) {
  return new Proxy(arr, {
    get(target, prop, receiver) {
      const index = Number(prop);
      if (Number.isInteger(index) && index < 0) {
        return target[target.length + index];
      }
      return Reflect.get(target, prop, receiver);
    },
  });
}

const arr = negativeArray([1, 2, 3, 4, 5]);
console.log(arr[-1]); // 5
console.log(arr[-2]); // 4
console.log(arr[0]);  // 1
```

### 4.2. Observable Object (Reactive)

```javascript
function observable(target, onChange) {
  return new Proxy(target, {
    set(obj, prop, value, receiver) {
      const oldValue = obj[prop];
      obj[prop] = value;
      
      if (oldValue !== value) {
        onChange({
          property: prop,
          oldValue,
          newValue: value,
          timestamp: Date.now(),
        });
      }
      
      return true;
    },
    
    deleteProperty(obj, prop) {
      const oldValue = obj[prop];
      delete obj[prop];
      
      onChange({
        property: prop,
        oldValue,
        newValue: undefined,
        type: "delete",
      });
      
      return true;
    },
  });
}

const state = observable({ count: 0, name: "John" }, (change) => {
  console.log(`Changed: ${change.property}: ${change.oldValue} → ${change.newValue}`);
});

state.count = 1;    // "Changed: count: 0 → 1"
state.name = "Jane"; // "Changed: name: John → Jane"
```

### 4.3. Immutable Object

```javascript
function immutable(target) {
  return new Proxy(target, {
    set() {
      throw new TypeError("Cannot modify immutable object");
    },
    deleteProperty() {
      throw new TypeError("Cannot delete from immutable object");
    },
    defineProperty() {
      throw new TypeError("Cannot define property on immutable object");
    },
  });
}

const frozen = immutable({ name: "John", age: 25 });
frozen.name;      // "John"
frozen.name = "Jane"; // ❌ TypeError
delete frozen.age;    // ❌ TypeError
```

### 4.4. API Rate Limiter

```javascript
function rateLimited(fn, maxCalls, windowMs) {
  const calls = [];
  
  return new Proxy(fn, {
    apply(target, thisArg, args) {
      const now = Date.now();
      
      // Remove old calls outside window
      while (calls.length > 0 && calls[0] < now - windowMs) {
        calls.shift();
      }
      
      if (calls.length >= maxCalls) {
        throw new Error(`Rate limit exceeded (${maxCalls} calls per ${windowMs}ms)`);
      }
      
      calls.push(now);
      return target.apply(thisArg, args);
    },
  });
}

const limitedFetch = rateLimited(fetch, 5, 1000); // Max 5 calls per second
```

### 4.5. Type-safe Object

```javascript
function typed(schema) {
  return new Proxy({}, {
    set(target, prop, value) {
      if (!(prop in schema)) {
        throw new Error(`Unknown property: ${prop}`);
      }
      
      const expectedType = schema[prop];
      if (typeof value !== expectedType) {
        throw new TypeError(
          `${prop} must be ${expectedType}, got ${typeof value}`
        );
      }
      
      target[prop] = value;
      return true;
    },
  });
}

const user = typed({
  name: "string",
  age: "number",
  active: "boolean",
});

user.name = "John";   // ✅
user.age = 25;         // ✅
user.active = true;    // ✅
user.age = "25";       // ❌ TypeError: age must be number
user.unknown = "x";    // ❌ Error: Unknown property
```

---

## 5. Reflect API

`Reflect` cung cấp các static methods tương ứng 1:1 với proxy traps. Là cách **chuẩn** để thực hiện các thao tác object mà trước đây phải dùng operators hoặc Object methods.

### 5.1. Tại sao dùng Reflect?

```javascript
// ❌ Cách cũ — mixed syntax
const obj = { name: "John" };
const val = obj.name;                    // Get
obj.age = 25;                             // Set
"name" in obj;                            // Has
delete obj.age;                           // Delete
Object.keys(obj);                         // Own keys
Object.getPrototypeOf(obj);              // Get prototype

// ✅ Reflect — consistent API
Reflect.get(obj, "name");                // Get
Reflect.set(obj, "age", 25);             // Set
Reflect.has(obj, "name");                // Has
Reflect.deleteProperty(obj, "age");      // Delete
Reflect.ownKeys(obj);                    // Own keys
Reflect.getPrototypeOf(obj);             // Get prototype
```

### 5.2. Reflect Methods

```javascript
const obj = { name: "John" };

// Property operations
Reflect.get(obj, "name");              // "John"
Reflect.set(obj, "age", 25);           // true
Reflect.has(obj, "name");              // true
Reflect.deleteProperty(obj, "age");    // true

// Property descriptor
Reflect.defineProperty(obj, "id", { value: 1 });
Reflect.getOwnPropertyDescriptor(obj, "name");

// All keys (string + symbol)
Reflect.ownKeys(obj);                 // ["name", "id"]

// Function operations
Reflect.apply(Math.max, null, [1, 2, 3]);     // 3
Reflect.construct(Date, [2024, 0, 1]);         // Date object

// Prototype
Reflect.getPrototypeOf(obj);
Reflect.setPrototypeOf(obj, null);

// Extensibility
Reflect.isExtensible(obj);              // true
Reflect.preventExtensions(obj);          // true
```

### 5.3. Reflect trong Proxy Handlers

**Best practice**: Dùng `Reflect` methods trong proxy handlers để forward behavior đúng cách.

```javascript
const handler = {
  // ❌ Cách cũ — có thể bỏ sót edge cases
  get(target, prop) {
    console.log(`Get: ${prop}`);
    return target[prop]; // Không handle receiver correctly
  },
  
  // ✅ Dùng Reflect — đúng và đầy đủ
  get(target, prop, receiver) {
    console.log(`Get: ${prop}`);
    return Reflect.get(target, prop, receiver); // Passes receiver correctly
  },
  
  set(target, prop, value, receiver) {
    console.log(`Set: ${prop} = ${value}`);
    return Reflect.set(target, prop, value, receiver);
  },
  
  has(target, prop) {
    console.log(`Has: ${prop}`);
    return Reflect.has(target, prop);
  },
  
  deleteProperty(target, prop) {
    console.log(`Delete: ${prop}`);
    return Reflect.deleteProperty(target, prop);
  },
};
```

### 5.4. Receiver Parameter

```javascript
// receiver quan trọng cho getter/setter inheritance
const parent = {
  _name: "parent",
  get name() {
    return this._name; // "this" cần là receiver
  },
};

const child = Object.create(parent);
child._name = "child";

const proxy = new Proxy(parent, {
  get(target, prop, receiver) {
    // ❌ target[prop] → this = parent → "parent"
    // ✅ Reflect.get(target, prop, receiver) → this = receiver → "child"
    return Reflect.get(target, prop, receiver);
  },
});
```

---

## 6. Revocable Proxy

```javascript
// Proxy có thể "tắt" (revoke)
const { proxy, revoke } = Proxy.revocable(
  { name: "John", secret: "classified" },
  {
    get(target, prop) {
      if (prop === "secret") {
        throw new Error("Access denied");
      }
      return target[prop];
    },
  }
);

proxy.name;   // "John"

// Revoke — proxy trở nên vô dụng
revoke();

proxy.name;   // ❌ TypeError: Cannot perform 'get' on a proxy that has been revoked
proxy.age = 25; // ❌ TypeError

// Use case: Temporary access, sessions, time-limited APIs
```

---

## 7. Câu hỏi phỏng vấn

### Q1: Proxy là gì?
**A**: Proxy là object wrapper intercept/customize các thao tác cơ bản (get, set, delete, ...) trên object gốc thông qua "traps" trong handler.

### Q2: Reflect khác gì Object methods?
**A**: Reflect methods tương ứng 1:1 với proxy traps, trả về boolean thay vì throw errors, và xử lý receiver parameter đúng cách. Là API chuẩn hóa thay thế cho operators và Object methods.

### Q3: Khi nào dùng Proxy?
**A**: Validation, logging, caching, access control, reactive programming, API wrappers, và bất kỳ khi nào cần customize object behavior mà không thay đổi object gốc.

---

> Quay lại [Mục lục](./README.md)
